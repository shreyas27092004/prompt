# System Design Deep Dive, Part 1: Foundations and Core Building Blocks

Every topic follows the same pattern: **What** it is, **Why** it exists, **How** it works, **Where** it's used, an **Example**, and the **Trade-offs**.

**Series:** Part 1 (this file) | Part 2: Data and Messaging | Part 3: Distributed Systems and Reliability | Part 4: Interview Framework and Case Studies

---

## 1. Client-Server Architecture

**What:** A model where a *client* (browser, mobile app, another service) sends requests and a *server* processes them and returns responses.

**Why:** Separating the UI from the logic and data lets many clients share one source of truth, lets you update the backend without redeploying every client, and lets you scale each side independently.

**How:**
1. Client builds a request (method, URL, headers, body).
2. Request travels over the network (TCP/TLS).
3. Server parses, authenticates, runs business logic, talks to the DB.
4. Server returns a status code and body.

**Where:** Every web app, mobile app, game, and API.

**Example:** A React app calls `GET /api/orders/42` on a Spring Boot server, which reads from MySQL and returns JSON.

```java
@RestController
@RequestMapping("/api/orders")
class OrderController {
    private final OrderService service;
    OrderController(OrderService service) { this.service = service; }

    @GetMapping("/{id}")
    ResponseEntity<OrderDto> get(@PathVariable long id) {
        return service.find(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
}
```

**Trade-offs:** Server becomes a bottleneck and single point of failure unless you replicate it. Network calls add latency and can fail, which the code must handle.

---

## 2. What Happens When You Type a URL (End-to-End)

**What:** The full journey of a request from keyboard to rendered page.

**Why it matters:** Every performance and reliability technique (CDN, caching, load balancing) attaches at a specific step. If you know the steps, you know where to optimize.

**How (step by step):**

| Step | What happens | Optimization that lives here |
|---|---|---|
| 1. Browser cache | Checks if the page/asset is cached | `Cache-Control`, `ETag` |
| 2. DNS lookup | Domain to IP | DNS caching, GeoDNS, low TTL for failover |
| 3. TCP handshake | SYN, SYN-ACK, ACK (1 RTT) | Keep-alive, HTTP/2, connection pooling |
| 4. TLS handshake | Negotiate keys (1-2 RTT) | TLS 1.3, session resumption |
| 5. HTTP request | Request goes to the nearest edge/CDN | CDN |
| 6. Load balancer | Picks a healthy server | LB algorithms, health checks |
| 7. App server | Business logic | Stateless design, async |
| 8. Cache lookup | Redis/Memcached | Cache-aside |
| 9. Database | Query | Indexes, replicas |
| 10. Response | Compressed, sent back | gzip/brotli, HTTP/2 |

**Where:** Interviews love this question. Real debugging uses it too: "Is slowness in DNS, TLS, server, or DB?" (Browser DevTools Network tab shows each phase.)

**Example:** A page takes 3 seconds. DevTools shows 200 ms DNS, 100 ms TLS, 2.5 s "waiting (TTFB)". The problem is server-side, so you look at DB queries, not the CDN.

---

## 3. DNS (Domain Name System)

**What:** A distributed, hierarchical database mapping human-readable names to IP addresses.

**Why:** Humans remember `shop.com`, machines need `203.0.113.10`. DNS also lets you change servers without telling users and route traffic by geography.

**How:**
1. Browser checks its cache, then the OS cache.
2. Asks the **recursive resolver** (your ISP or 8.8.8.8).
3. Resolver asks a **root server**, then the **TLD server** (`.com`), then the **authoritative server** for `shop.com`.
4. Answer is cached according to the record's **TTL**.

**Record types:**

| Record | Purpose | Example |
|---|---|---|
| A | name to IPv4 | `shop.com -> 203.0.113.10` |
| AAAA | name to IPv6 | |
| CNAME | alias to another name | `www.shop.com -> shop.com` |
| MX | mail servers | |
| NS | authoritative nameservers | |
| TXT | verification, SPF, DKIM | |

**Where:** Everywhere. As a design tool: **GeoDNS** (users in India get the Mumbai IP), **weighted DNS** (10% of traffic to the new version), **failover DNS** (health-check-driven).

**Example:** Before a migration, lower TTL from 24 h to 60 s a day ahead. At cutover change the A record; within about a minute nearly all clients follow. Raise TTL again afterward.

**Trade-offs:** Low TTL means faster failover but more DNS queries. Some resolvers ignore low TTLs, so DNS failover is never instant.

---

## 4. Networking Protocols: TCP, UDP, HTTP, WebSocket

### TCP vs UDP

**What:** TCP is reliable and ordered (handshake, retransmits, congestion control). UDP just fires packets with no guarantees.

**Why both exist:** Reliability costs latency. Some data is useless if late (a video frame), so retransmitting it is wasteful.

**Where:**
- TCP: HTTP(S), databases, SSH, email.
- UDP: DNS, live video/voice, online games, QUIC (HTTP/3).

### HTTP versions

| Version | Key idea | Problem solved |
|---|---|---|
| 1.1 | Persistent connections | Reopening TCP per request |
| 2 | Multiplexing many requests on one connection, header compression | Head-of-line blocking at HTTP level |
| 3 | Runs on QUIC (UDP) | Head-of-line blocking at TCP level, faster connection setup |

### WebSocket

**What:** A persistent, two-way channel upgraded from an HTTP connection.
**Why:** HTTP is request-response; the server can't push. Polling wastes bandwidth.
**How:** Client sends `Upgrade: websocket`; after the handshake both sides send frames anytime.
**Where:** Chat, live scores, collaborative editing, trading dashboards.
**Example:** Chat server keeps 100k open sockets per node; each message is pushed instantly to the recipient's socket.
**Trade-offs:** Long-lived connections are stateful, which complicates load balancing and deploys (need connection draining and reconnection logic).

### Real-time options compared

| Technique | Direction | Use when |
|---|---|---|
| Short polling | Client pulls every N s | Simple, low-frequency updates |
| Long polling | Client pulls, server holds until data | WebSocket not available |
| SSE (Server-Sent Events) | Server to client only | Notifications, live feeds, progress bars |
| WebSocket | Both ways | Chat, games, collaboration |

---

## 5. API Styles: REST, GraphQL, gRPC

### REST

**What:** Resource-oriented API over HTTP using standard verbs.
**Why:** Simple, cacheable, universally supported, human-readable.
**How:**

| Verb | Meaning | Idempotent? | Example |
|---|---|---|---|
| GET | Read | Yes | `GET /users/42` |
| POST | Create / action | No | `POST /orders` |
| PUT | Replace | Yes | `PUT /users/42` |
| PATCH | Partial update | Usually not guaranteed | `PATCH /users/42` |
| DELETE | Remove | Yes | `DELETE /users/42` |

**Status codes to know:** 200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthenticated, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Too Many Requests, 500 Internal Error, 502/503/504 gateway/unavailable/timeout.

**Pagination:**
- **Offset:** `?page=100&size=20` becomes `OFFSET 2000`, so the DB scans and discards 2000 rows. Slow deep in the list, and results shift if data changes.
- **Cursor/keyset:** `?after=1042&limit=20` becomes `WHERE id > 1042 ORDER BY id LIMIT 20`. Constant time using the index; stable.

**Example:**
```sql
-- Offset (slow at depth)
SELECT * FROM posts ORDER BY id LIMIT 20 OFFSET 100000;
-- Keyset (fast)
SELECT * FROM posts WHERE id > 100000 ORDER BY id LIMIT 20;
```

### GraphQL

**What:** One endpoint; the client asks for exactly the fields it needs.
**Why:** Fixes over-fetching (too much data), under-fetching (multiple round trips), and fits many different client screens.
**Where:** Mobile apps with many views, GitHub API, Shopify.
**Trade-offs:** HTTP caching is harder, N+1 query problem on the server (use DataLoader batching), need query depth/complexity limits to prevent abusive queries.

### gRPC

**What:** Binary RPC using Protocol Buffers over HTTP/2, with generated typed clients and streaming.
**Why:** Smaller payloads, faster serialization, strict contracts, bi-directional streaming.
**Where:** Internal microservice-to-microservice calls.
**Trade-offs:** Not browser-friendly without a proxy, binary is harder to debug.

**Decision rule:** Public API means REST. Flexible client data needs mean GraphQL. Internal high-throughput service calls mean gRPC.

---

## 6. The Vocabulary of Performance and Reliability

### Latency, throughput, percentiles

**What:** Latency is time per request. Throughput is requests per second. Percentiles (p50, p95, p99) describe the distribution.
**Why percentiles matter:** The average hides pain. If p99 is 5 s, 1 in 100 users waits 5 s; with 100 requests per page load, most page loads hit at least one slow call.
**Example:** Average 100 ms but p99 3 s. Look for GC pauses, cold caches, lock contention, noisy neighbors.

### Availability

`Availability = uptime / (uptime + downtime)`

| Target | Allowed downtime / year |
|---|---|
| 99.9% | 8.76 h |
| 99.99% | 52 min |
| 99.999% | 5 min |

**Serial dependencies multiply:** A (99.9%) calls B (99.9%): 0.999 x 0.999 is about 99.8%. Ten serial dependencies at 99.9% each gives about 99%. This is why long synchronous call chains in microservices hurt availability.
**Parallel redundancy improves it:** two independent 99% nodes give 1 - (0.01 x 0.01) = 99.99%, *if failures are truly independent*.

### SLI, SLO, SLA, error budget

- **SLI:** the measurement (e.g. % of requests under 300 ms).
- **SLO:** the internal target (99.9% of requests under 300 ms per month).
- **SLA:** the contract with penalties (refund if below 99.5%).
- **Error budget:** 100% minus SLO. If you've burned the budget, freeze risky releases and fix reliability.

---

## 7. Scaling: Vertical vs Horizontal

**What:** Vertical = bigger machine. Horizontal = more machines.

**Why it matters:** One machine has a hard ceiling and is a single point of failure.

**How horizontal scaling works:**
1. Make the app **stateless** (no user data in server memory).
2. Put a **load balancer** in front.
3. Move state to shared stores (DB, Redis).
4. Add or remove instances based on load (autoscaling).

**Where:** Vertical is fine early on and for databases (up to a point). Horizontal is the default for web/app tier.

**Example (making it stateless):**
- Bad: login session stored in a `HashMap` inside the JVM. A second server doesn't know the user.
- Good: JWT in the client, or session stored in Redis (Spring Session).

```java
// Spring Session with Redis: sessions are shared by every instance
@EnableRedisHttpSession
class SessionConfig {}
```

**Trade-offs:** Horizontal scaling shifts complexity to distributed data (consistency, coordination). Vertical is simpler but capped and expensive at the top end.

---

## 8. Load Balancer

**What:** A component that spreads incoming requests across multiple backend servers.

**Why:** Prevents any one server from being overloaded, removes the single point of failure, and enables zero-downtime deploys (drain one instance, update, return).

**How:**
1. LB holds a pool of backends and periodically **health checks** them (`GET /health`).
2. For each request, an **algorithm** picks a backend:
   - **Round robin:** A, B, C, A, B, C. Simple; assumes equal requests/servers.
   - **Weighted round robin:** bigger servers get more.
   - **Least connections:** to whoever has the fewest active requests; good for long-lived or uneven requests.
   - **IP hash / consistent hash:** same client always goes to the same server (stickiness, cache locality).
3. Unhealthy backends are removed until they recover.

**L4 vs L7:**

| | L4 (transport) | L7 (application) |
|---|---|---|
| Sees | IP, port | URL, headers, cookies |
| Speed | Faster | Slightly slower |
| Features | Basic balancing | Path routing (`/api` vs `/static`), TLS termination, auth, rewrites, A/B |
| Examples | AWS NLB, HAProxy TCP | Nginx, AWS ALB, Envoy |

**Where:** In front of every horizontally scaled tier; also between services (internal LB), and globally (DNS/anycast to pick a region).

**Example (Nginx):**
```nginx
upstream app {
    least_conn;
    server 10.0.0.11:8080 max_fails=3 fail_timeout=10s;
    server 10.0.0.12:8080;
    server 10.0.0.13:8080 backup;
}
server {
    listen 443 ssl;
    location /api/    { proxy_pass http://app; }
    location /static/ { root /var/www; expires 7d; }
}
```

**Trade-offs and gotchas:**
- The LB itself must be redundant (active-passive pair or managed service).
- **Sticky sessions** make scaling and failover harder; prefer stateless servers.
- Health checks must test real readiness (dependencies reachable), not just "process is up", or you'll send traffic to a broken node.

---

## 9. Reverse Proxy, Forward Proxy, API Gateway

**What:**
- **Forward proxy:** acts for the *client* (corporate web filter, VPN).
- **Reverse proxy:** acts for the *server* (hides backends, terminates TLS, caches, compresses).
- **API gateway:** a reverse proxy specialized for APIs: auth, rate limiting, routing, aggregation, monitoring.

**Why a gateway:** Without it, every microservice re-implements authentication, throttling, logging, CORS. Clients would need to know every service address.

**How (request through a gateway):**
1. Client sends request to `api.shop.com`.
2. Gateway validates JWT, applies rate limit, adds trace ID.
3. Routes `/orders/**` to Order Service, `/users/**` to User Service.
4. Optionally aggregates responses or transforms protocols.

**Where:** Any microservice architecture. Examples: Kong, AWS API Gateway, Spring Cloud Gateway, Apigee.

**Example (Spring Cloud Gateway):**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: orders
          uri: lb://order-service
          predicates: [ Path=/orders/** ]
          filters:
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

**Trade-offs:** Gateway is a critical path component (must be highly available) and can become a "god" layer if business logic creeps in. Keep it thin.

---

## 10. Caching (In Depth)

### 10.1 What and why

**What:** Storing copies of frequently-used data in a faster layer.
**Why:** RAM is about 1000x faster than SSD and far faster than a network round trip to a DB. Caching cuts latency, reduces DB load, and saves cost.
**When it works:** Read-heavy data, expensive computations, data that changes rarely (product catalog, config, user profile, leaderboards).
**When it doesn't:** Write-heavy, highly unique per-request data, data requiring strict real-time accuracy (account balance on a payment decision).

### 10.2 Where caches live (layers)

| Layer | Example | What it caches |
|---|---|---|
| Browser | `Cache-Control: max-age=3600` | Static files, API GETs |
| CDN | CloudFront, Cloudflare | Images, JS/CSS, video, some API responses |
| Reverse proxy | Nginx | Full HTTP responses |
| App (in-process) | Caffeine, Guava | Small hot data; zero network hop |
| Distributed | Redis, Memcached | Shared across instances |
| DB | Buffer pool, query cache | Pages and plans |

### 10.3 Strategies

**Cache-aside (lazy loading):**
1. App checks cache.
2. Miss: reads DB.
3. Writes result to cache with TTL.
4. Returns data.
On update: update DB, then **delete** the cache key (don't update it, to avoid races).

```java
public Product getProduct(long id) {
    String key = "product:" + id;
    Product cached = redis.get(key);
    if (cached != null) return cached;

    Product p = repo.findById(id).orElseThrow();
    redis.set(key, p, Duration.ofMinutes(10));
    return p;
}

@Transactional
public void updateProduct(Product p) {
    repo.save(p);
    redis.delete("product:" + p.getId());
}
```
Or declaratively in Spring: `@Cacheable("products")` and `@CacheEvict("products")`.

**Write-through:** write to cache and DB together. Reads always fresh; writes slower.
**Write-behind:** write to cache, flush to DB asynchronously. Fast writes; risk of data loss if cache dies before flush. Use for counters, analytics, not money.
**Read-through:** like cache-aside but the cache library itself loads from the DB.
**Write-around:** write to DB only; cache fills on next read. Good when written data is rarely read soon.

### 10.4 Eviction policies

- **LRU:** evict least recently used (most common).
- **LFU:** evict least frequently used (good for stable hot sets).
- **TTL:** expire after a fixed time (always set one).
- **FIFO / random:** simple, rarely best.

### 10.5 Classic cache failures and fixes

| Problem | What happens | Fix |
|---|---|---|
| **Stampede / thundering herd** | A hot key expires, 10,000 requests hit the DB at once | Request coalescing (one loader, others wait), mutex/lock per key, probabilistic early refresh, serve-stale-while-revalidate |
| **Penetration** | Requests for non-existent keys always miss and hit DB (attack or bug) | Cache nulls briefly, Bloom filter in front, validate input |
| **Avalanche** | Many keys expire at the same instant | Add random jitter to TTLs; warm up gradually |
| **Hot key** | One key gets huge traffic and overloads one Redis node | Local in-process cache, replicate key as `key#1..N`, split reads across replicas |
| **Stale data** | Cache differs from DB | Short TTL, delete-on-write, event-driven invalidation |
| **Cache as source of truth** | Cache wiped and data is gone | Always be able to rebuild from DB |

**Example (jittered TTL):**
```java
Duration ttl = Duration.ofMinutes(10).plusSeconds(ThreadLocalRandom.current().nextInt(0, 120));
```

### 10.6 Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Data types | Strings, hashes, lists, sets, sorted sets, streams, geo, HyperLogLog | Strings only |
| Persistence | Optional (RDB/AOF) | None |
| Replication / clustering | Yes | Client-side sharding |
| Threading | Mostly single-threaded command execution | Multi-threaded |
| Use | Cache + queues + rate limits + leaderboards + locks + pub/sub | Pure simple cache |

**Example uses of Redis data types:**
- Leaderboard: sorted set (`ZADD scores 1500 alice`, `ZREVRANGE scores 0 9`).
- Rate limit: `INCR` + `EXPIRE`.
- Session: hash with TTL.
- Unique visitors: HyperLogLog.
- Nearby drivers: GEO commands.

### 10.7 What to cache and for how long

- Cache **IDs and small objects**, not giant blobs.
- TTL by tolerance for staleness: product description (hours), inventory count (seconds or no cache), user permissions (minutes, invalidate on change).
- Measure the **hit ratio**. Below about 80% for a cache you expect to be effective means something's wrong (TTL too short, key design, too small).

---

## 11. CDN (Content Delivery Network)

**What:** A globally distributed network of edge servers that cache content near users.

**Why:** Distance means latency (cross-continent is about 150 ms round trip). A CDN serves from a nearby edge, cutting latency, offloading your origin, and absorbing traffic spikes and DDoS.

**How:**
1. DNS directs the user to the nearest edge (GeoDNS or anycast).
2. Edge has the file? **Hit**: return immediately.
3. **Miss**: edge fetches from origin, stores per `Cache-Control`, returns.

**Pull vs Push:**
- **Pull:** CDN fetches on first request. Easy; first user is slow.
- **Push:** you upload ahead of time. Good for big, predictable content (software releases, video).

**Cache invalidation:** Purging is slow and unreliable at scale. Best practice: **fingerprinted filenames** (`app.8f3a2c.js`) with `Cache-Control: max-age=31536000, immutable`. A new deploy produces a new filename, so there's nothing to invalidate. Keep `index.html` short-TTL.

**Where:** Static assets (JS, CSS, images), video streaming, software downloads, and increasingly dynamic API caching and edge compute.

**Example:** A Mumbai user loads a React bundle from a Mumbai edge in 20 ms instead of a Virginia origin in 250 ms. Your origin sees 1 request per file per edge, not millions.

**Trade-offs:** Stale content if TTL and invalidation are mismanaged; cost per GB egress; private content needs signed URLs/cookies.

---

## 12. Stateless vs Stateful Services

**What:** A *stateless* service keeps no client-specific data between requests. A *stateful* one does (sessions, in-memory caches, open sockets, data on local disk).

**Why it matters:** Stateless instances are interchangeable, so you can scale, replace, and load-balance freely. Stateful components need careful handling (replication, partitioning, sticky routing).

**How to make a service stateless:**
- Sessions: JWT or external session store.
- Files: object storage (S3), not local disk.
- Cache: Redis, not per-instance maps (or accept per-instance caches as just an optimization).
- Config: environment variables / config service.

**Where stateful is unavoidable:** Databases, caches, message brokers, WebSocket servers, game servers. Handle with replication, partitioning, and StatefulSets in Kubernetes.

**Example:** Upload service writes to `/tmp/uploads`. After deploy, a different pod handles the "download" request and can't find the file. Fix: upload directly to S3 using pre-signed URLs.

---

## 13. Putting Part 1 Together: A Starter Architecture

```
            Users
              |
            [DNS]  (GeoDNS)
              |
            [CDN]  (static: JS/CSS/images)
              |
        [Load Balancer]  (TLS termination)
         /     |     \
   [App 1] [App 2] [App 3]   (stateless Spring Boot, Docker)
         \     |     /
      [Redis cache / sessions]
              |
        [MySQL primary] --replication--> [MySQL replica x2]
```

**Request path (read):** CDN for static; LB picks app; app checks Redis; on miss reads from a replica; fills cache.
**Request path (write):** App writes to the primary; invalidates the cache key; replica gets the change asynchronously.

**What this design already gives you:** no single app server SPOF, fast static delivery, cached reads, read scaling. **What's still missing** (covered in later parts): async work via queues, DB sharding, multi-region, resilience patterns, observability.
