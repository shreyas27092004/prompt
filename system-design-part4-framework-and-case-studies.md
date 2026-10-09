# System Design Deep Dive, Part 4: Framework, Estimation and Case Studies

**Series:** Part 1: Foundations | Part 2: Data and Messaging | Part 3: Distributed Systems and Reliability | Part 4 (this file)

---

## 1. The Design Framework (Use This Every Time)

**What:** A repeatable sequence for tackling any "Design X" problem, in an interview or at work.

**Why:** Open-ended problems overwhelm people. A framework keeps you structured, shows your reasoning, and prevents jumping to technology before understanding the problem.

### Step 1: Clarify requirements (about 5 min)

Ask questions. Split into:

**Functional (what it does):** Core features only. "Users can post, follow, view feed" (not "also stories, reels, DMs"). State what's **out of scope**.

**Non-functional (how well):**
- Scale: DAU, requests/sec, data volume, growth.
- Latency target (e.g. feed loads under 200 ms p99).
- Availability (99.9%? 99.99%?).
- Consistency needs (strong or eventual per feature).
- Durability (can we ever lose data?).
- Geography (single region or global?).
- Security/compliance.

### Step 2: Back-of-envelope estimation (about 5 min)

**Why:** Numbers decide the architecture. 100 QPS needs one server; 100,000 QPS needs a distributed design. 1 GB of data fits in memory; 1 PB needs sharding.

**Method:**
1. Users, DAU, actions per user per day.
2. QPS = (DAU x actions/day) / 86,400. Peak = 2x to 5x average.
3. Storage = records/day x size x retention.
4. Bandwidth = QPS x payload size.
5. Cache memory = hot working set (often 20% of daily reads).

**Worked example (photo sharing):**
```
DAU = 20M. Each views 50 photos/day, uploads 0.2 photos/day.
Reads  = 20M x 50  = 1B/day   -> 1e9 / 86,400  ~ 11,600 QPS avg, ~35k peak
Writes = 20M x 0.2 = 4M/day   ->                ~ 46 QPS avg, ~150 peak
Read:write ratio = 250:1  => heavily read-optimized (CDN, cache, replicas)
Storage: 4M photos/day x 2 MB = 8 TB/day => ~2.9 PB/year => object storage, tiering
Egress: 35k QPS x 200 KB (thumbnails) = 7 GB/s peak => CDN is mandatory
```
Conclusions from numbers: CDN is essential, metadata DB is small (tens of GB/day), photos live in object storage, uploads are low-QPS so a simple upload path is fine.

**Memory aids:**
- 1 day = 86,400 s (about 10^5). 1M req/day is about 12 QPS. 1B req/day is about 12k QPS.
- 1 KB = 10^3, 1 MB = 10^6, 1 GB = 10^9, 1 TB = 10^12.
- One server: thousands to tens of thousands of simple QPS; Redis about 100k ops/s; a MySQL node handles low thousands of QPS on typical queries.

### Step 3: API design (about 3 min)

List key endpoints with params and responses. Decide REST vs gRPC, pagination style, auth.
```
POST /v1/photos            {caption, visibility}  -> {photoId, uploadUrl}
GET  /v1/feed?cursor=abc   -> {items[], nextCursor}
POST /v1/users/{id}/follow
```

### Step 4: Data model and storage choice (about 5 min)

Entities, relationships, **access patterns** (what queries?), choice of DB(s) with reasons, partition keys, indexes.

### Step 5: High-level design (about 10 min)

Draw boxes and arrows: clients, CDN, LB, API gateway, services, cache, DBs, queues, workers, object storage. Walk through the **write path** and **read path** for the main flows.

### Step 6: Deep dives (about 10 min)

Pick the hardest 2 to 3 problems (or follow the interviewer): sharding, feed generation, hot keys, consistency, ID generation, search, real-time delivery.

### Step 7: Bottlenecks, failures, trade-offs (about 5 min)

- Single points of failure? Replicas, multi-AZ.
- What if X is down? Fallbacks, retries, degradation.
- 10x growth? What breaks first?
- Monitoring and alerts. Security. Cost.
- **Explicitly state trade-offs** ("I chose eventual consistency for feeds because...").

### Interview behavior tips

- Think aloud; ask clarifying questions; summarize assumptions.
- Start simple (one server), then evolve it as numbers demand.
- Justify each component by a requirement. No buzzword dumping.
- Prefer proven, boring tech unless the requirement forces otherwise.
- Acknowledge alternatives and why you didn't choose them.

---

## 2. Case Study: URL Shortener

### Requirements
- **Functional:** create short URL from long URL (optional custom alias, expiry); redirect short to long; basic click analytics.
- **Non-functional:** very low redirect latency (under 50 ms), 99.99% availability for redirects, links not guessable (optional), durable.
- **Out of scope:** user accounts UI, QR codes.

### Estimation
```
100M new URLs/month  -> ~40 writes/s
Read:write 100:1     -> ~4,000 reads/s (peak ~12k)
Record ~500 B x 100M x 12 months x 5 years = ~3 TB
```
Small data, high read QPS, so cache and replicas matter more than sharding at first.

### API
```
POST /v1/shorten {longUrl, alias?, expiresAt?} -> 201 {shortUrl}
GET  /{code}                                    -> 302 Location: longUrl
```
**301 vs 302:** 301 (permanent) is cached by browsers, saving your servers but you lose click counts. Use **302** if you need analytics.

### Key generation: how to make the 7-char code
Base62 (a-z, A-Z, 0-9). 62^7 is about 3.5 trillion combinations, enough for decades.

| Approach | How | Pros | Cons |
|---|---|---|---|
| Hash (MD5/SHA, take first 7 chars) | Deterministic | Same URL gives same code | Collisions; need retry/check |
| **Counter/ID plus base62** | Unique ID (Snowflake, DB range) encoded | No collisions, simple | Sequential and guessable (mitigate by shuffling bits/encrypting the ID) |
| Pre-generated key service (KGS) | Offline pool of unused keys; app takes a batch | No runtime collisions; fast | Extra service; must not hand out the same key twice |

**Choice:** Counter-based with ID ranges: each app server reserves a block of 1000 IDs from a coordination service and base62-encodes locally.

```java
static final String ALPHABET = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
static String toBase62(long n) {
    StringBuilder sb = new StringBuilder();
    do { sb.append(ALPHABET.charAt((int)(n % 62))); n /= 62; } while (n > 0);
    return sb.reverse().toString();
}
```

### Data model
```
urls(code PK, long_url, user_id, created_at, expires_at)
```
Access pattern is pure key lookup, so a KV store (DynamoDB/Cassandra) or sharded MySQL by `code` works; both scale. MySQL is fine at 3 TB with replicas.

### Architecture
```
Client -> CDN/Edge -> LB -> Redirect Service -> Redis (code -> url) -> DB (replicas)
                              |
                              +--> Kafka (click events) -> Analytics workers -> warehouse
Create path: LB -> Shorten Service -> ID allocator -> DB primary (+ cache write)
```

### Deep dives
- **Caching:** Redis with LRU; the top 20% of links get 80% of traffic. TTL aligned with expiry. Redirect path should hit cache more than 95% of the time.
- **Custom alias uniqueness:** DB unique constraint on `code` (source of truth); Bloom filter as a fast pre-check.
- **Expiry cleanup:** lazy deletion on read plus a background sweeper job.
- **Analytics:** don't write to the DB on every click (that would add DB write load of 4k/s per redirect). Publish click events to Kafka, aggregate asynchronously.
- **Abuse:** rate-limit creation per user/IP; scan for malicious destinations.
- **Availability:** multiple stateless redirect servers across AZs; DB replicas; cache failure just increases DB load (so size the DB to survive a cold cache or use request coalescing).

### Trade-offs
302 vs 301 (analytics vs load); sequential IDs (guessability) vs random hash (collisions); strong vs eventual consistency (creation can be strongly consistent; analytics eventual).

---

## 3. Case Study: Rate Limiter Service

### Requirements
Limit requests per client by rule (e.g. 100/min per API key), low overhead (under 5 ms added), works across many servers, clear client feedback.

### Where it lives
At the **API gateway** (central), or as a **sidecar/middleware** in each service. Edge-level IP limits live in the CDN/WAF.

### Design
1. Request arrives at gateway, which extracts the key (API key, user ID, IP).
2. Looks up the **rule** (cached from a rules store).
3. Calls the **Redis** counter/token bucket (atomic Lua script).
4. Allowed means forward; denied means `429` with `Retry-After`.

**Algorithm choice:** token bucket (supports bursts, tiny memory: two numbers per key).

### Deep dives
- **Race conditions:** two servers read-increment-write at once. Solve with atomic Redis operations (`INCR`, Lua).
- **Redis as a bottleneck/SPOF:** shard keys across a Redis cluster; replicas; decide **fail-open** (allow if the limiter is down: protects availability) vs **fail-closed** (block: protects security endpoints like login).
- **Latency:** use local in-memory pre-checks plus periodic sync for very high-traffic keys (approximate but fast).
- **Multi-region:** per-region limits (simple) or global with eventual sync (approximate).
- **Rule updates:** push via config service; cache with short TTL.
- **Fairness and tiers:** different limits per plan; separate buckets per endpoint (expensive endpoints get stricter limits).

---

## 4. Case Study: Chat System (WhatsApp / Slack style)

### Requirements
- **Functional:** 1:1 and group chat, send/receive in real-time, delivery and read receipts, online presence, message history, media, push notifications for offline users.
- **Non-functional:** low latency (under 200 ms), messages **never lost**, **ordered within a conversation**, highly available, scale to hundreds of millions of users.

### Estimation
```
500M DAU x 40 messages/day = 20B messages/day -> ~230k msgs/s avg (peak ~700k)
Message ~100 B (text) => 2 TB/day text; media is separate (object storage)
Concurrent connections: say 100M open sockets => at 100k/server => ~1,000 chat servers
```

### Core design

**Connection layer:** Clients keep a **WebSocket** to a **chat gateway server**. Because connections are long-lived and stateful:
- LB uses connection-level balancing (not per-request).
- A **session registry** (Redis) maps `userId -> gatewayServerId` (with TTL refreshed by heartbeats).

**Send flow (1:1):**
1. Alice's client sends the message over her socket to gateway G1 with a client-generated `messageId` (for idempotency).
2. G1 forwards to the **Chat Service**, which:
   - assigns a **per-conversation sequence number**,
   - **persists** the message (write-ahead; the message is durable before ack),
   - acks to Alice ("sent").
3. Chat Service looks up Bob in the session registry.
   - **Online** on G2: push to G2 (via internal RPC or pub/sub), which writes to Bob's socket; Bob's client acks ("delivered").
   - **Offline:** enqueue a **push notification** (APNs/FCM); message waits in storage; on reconnect Bob's client says "my last seen seq in conversation X is 118" and the server sends 119+.
4. When Bob reads, "read" receipt flows back the same way.

**Why per-conversation sequence numbers, not timestamps:** clocks differ across servers; sequence numbers give a total order inside a conversation. Clients detect gaps (seq 5 then 7) and fetch the missing one.

### Storage
- **Messages:** wide-column store (Cassandra/HBase/ScyllaDB), partition key = `conversationId`, clustering key = `seq`. Writes are append-heavy and reads are "last N messages of this chat", a perfect fit.
- **Users, groups, membership:** relational DB (MySQL/Postgres).
- **Media:** object storage with pre-signed uploads, CDN delivery; message stores only a reference and thumbnail.
- **Hot recent messages / unread counts:** Redis.

### Group chat
- **Small groups (under about 100):** fan-out on write: deliver to each member's connection/inbox.
- **Large channels (thousands+):** store once; members pull on open, with push only for active/subscribed viewers (avoids write amplification).
- Membership lookups cached.

### Presence (online/last seen)
- Heartbeat every ~5 s sets a Redis key with ~15 s TTL.
- Don't broadcast every change to everyone. Publish status only to a user's active contacts, or let clients **pull** presence for visible contacts (lazy). Otherwise 100M users toggling generate a notification storm.

### Reliability and edge cases
- **Idempotency:** client `messageId` dedupes retries.
- **Multi-device:** each device has its own connection and sync cursor; sequence per conversation, cursor per device.
- **Gateway crash:** clients reconnect (with jittered backoff) to another gateway and resync from last seq.
- **E2E encryption** (Signal protocol): servers relay ciphertext; key exchange via prekeys; complicates server-side search and multi-device.

### Trade-offs
Fan-out on write (fast reads, expensive for big groups) vs on read; strong ordering per conversation (via a single sequencer per conversation, which is also a scaling unit) vs global ordering (unnecessary); WebSocket statefulness vs simplicity of polling.

---

## 5. Case Study: News Feed (Twitter / Instagram)

### Requirements
Post content, follow users, see a feed of followed users' posts (ranked/recent), fast load (under 200 ms), tolerate slight staleness.

### Estimation
```
300M DAU, avg 10 feed loads/day => 3B reads/day => ~35k QPS (peak ~100k)
Posts: 50M/day => ~600 writes/s
Follower distribution is skewed: average 200, celebrities 50M+
```
Read-heavy and highly skewed. The central design question is **when to do the work**.

### Fan-out strategies

**Fan-out on write (push):**
- When a user posts, a worker writes the post ID into **each follower's feed list** (Redis sorted set/list; also persisted).
- Read = fetch the precomputed list (very fast).
- Cost: a user with 50M followers causes 50M writes per post (**write amplification**, latency, wasted work for inactive followers).

**Fan-out on read (pull):**
- On feed load, fetch recent posts from everyone you follow and merge.
- Cheap writes; slow reads (follow lists can be large), hard to meet latency.

**Hybrid (what real systems do):**
- Normal users: **push**.
- Celebrities (above a follower threshold): **no fan-out**; at read time, merge the precomputed feed with recent posts of the few celebrities the viewer follows.
- Only fan out to **active** users (skip people inactive for 30 days).

### Architecture
```
Post Service -> DB (posts, sharded by userId or postId) + Kafka "post.created"
Fan-out workers (consume) -> look up followers (Graph Service) -> write postIds to feed cache
Feed Service: read feed IDs (Redis) + merge celebrity posts -> hydrate post/user data (cache) -> rank -> return
Media: object storage + CDN
```

### Data
- **Posts:** `post_id (Snowflake), user_id, content, media_refs, created_at` in a sharded DB (or Cassandra).
- **Social graph:** `follower_id, followee_id` indexed both ways; sharded; cached for hot users.
- **Feed cache:** per-user capped list (e.g. latest 800 post IDs). Store **IDs only**, then **hydrate** from post and user caches (keeps memory small and always shows fresh edits/deletes).

### Deep dives
- **Pagination:** cursor-based (post ID/time), never offset.
- **Ranking:** start chronological; later ML ranking with features (affinity, recency, engagement) in a ranking service; candidate generation then ranker.
- **Deletes/edits:** feed holds IDs, so hydration returns the latest state; deleted posts are filtered.
- **Hot posts:** cache aggressively, replicate hot keys.
- **Consistency:** eventual is fine (a post may take a few seconds to appear), but give the author **read-your-writes** (insert own post into own feed immediately).
- **Failure:** if fan-out lags, the queue absorbs it; feed reads still work with older data (graceful degradation).

---

## 6. Case Study: Video Streaming (YouTube / Netflix)

### Requirements
Upload, process, store, and stream videos globally with smooth playback on any device and network; search and recommendations.

### Upload and processing pipeline
1. Client requests an upload session; API returns **pre-signed multipart/resumable upload URLs**.
2. Client uploads chunks **directly to object storage** (no load on app servers; resumable on failure).
3. Upload-complete event goes to a **queue**.
4. **Transcoding pipeline** (workers, often a DAG):
   - Split the video into small segments (a few seconds each) for parallel processing.
   - Encode each into multiple **resolutions and bitrates** (240p to 4K) and codecs (H.264, VP9, AV1).
   - Generate thumbnails, subtitles, audio tracks, content moderation checks.
   - Package as **HLS/DASH** segments plus manifest.
5. Store outputs in object storage; update metadata DB; mark "ready"; notify the uploader.

### Playback: adaptive bitrate streaming
- The manifest lists renditions; the player downloads segments (2 to 10 s each) and **switches quality dynamically** based on measured bandwidth and buffer.
- Segments are plain HTTP files, so they are **CDN-cacheable**, which is why streaming scales.
- Popular content is **pre-positioned** on edge caches (Netflix Open Connect appliances sit inside ISPs at night).

### Components
- **Metadata DB** (title, owner, status, renditions): relational or NoSQL.
- **Search:** Elasticsearch fed by events.
- **Recommendations:** offline training plus online ranking.
- **View counts/likes:** stream aggregation (Kafka to Flink) with approximate, eventually consistent counters; periodic flush to DB.
- **Access control/DRM:** signed URLs, token auth, Widevine/FairPlay.

### Scale and cost
- Storage and egress dominate cost. Use **tiered storage** (hot/warm/cold), store fewer renditions for rarely watched videos, encode popular videos at more efficient (costlier-to-encode) settings.
- Transcoding is bursty, so use autoscaling worker fleets (spot instances) driven by queue depth.
- Long-tail vs head: a few videos get most views, so CDN hit ratio is high for them; the tail hits origin.

---

## 7. Case Study: Ride Hailing (Uber)

### Requirements
Riders request rides; nearby drivers receive offers; real-time tracking; pricing; payments; trip history. Low-latency matching, high location-update write rate.

### Estimation
```
1M drivers online x location update every 4 s => 250k writes/s
Ride requests: ~1,000/s at peak
```
The location stream is the load driver.

### Core pieces

**Location ingestion:**
- Drivers' apps send GPS pings over persistent connections to a **Location Service**.
- Keep the **latest location** per driver in an **in-memory geospatial index** (Redis GEO, or custom index using geohash/S2/H3 cells). History goes to Kafka then a time-series/columnar store for analytics and ETA models.

**Geospatial indexing (why it's needed):** "Find drivers within 2 km" can't scan all drivers. Split the map into **cells** (geohash/H3/quadtree). A driver belongs to one cell. A query looks at the rider's cell plus neighbors only.

**Matching:**
1. Rider requests a ride; the **Matching Service** queries nearby available drivers by cell.
2. Rank candidates by ETA (routing service), rating, and other factors.
3. Offer to the top driver with a timeout (about 10 s); on decline/timeout, offer to the next (or broadcast to a few).
4. On accept: create the trip, mark the driver **unavailable**. Use an atomic state transition so one driver can't be assigned two rides and one ride can't be assigned to two drivers.

**Trip service:** a strongly consistent **state machine** (REQUESTED, MATCHED, DRIVER_ARRIVED, IN_PROGRESS, COMPLETED, CANCELLED). Stored in a sharded DB keyed by trip ID; events published for notifications, pricing, and analytics.

**ETA/routing:** road network graph, shortest path with live traffic (contraction hierarchies/A*), ML adjustments.

**Surge pricing:** stream processing counts supply and demand per cell per minute; multiplier applied by the pricing service; smooth across neighboring cells to avoid cliff effects.

**Payments:** a saga: authorize at request, capture at completion; **idempotency keys**; ledger; reconciliation jobs.

**Real-time updates:** WebSocket/push to rider and driver for location and status.

### Scaling
- Shard by **geography/city**: matching is local, so cities are naturally independent units (and can run in the nearest region).
- Kafka for event backbone; services stay stateless apart from the in-memory geo index (which is rebuildable from the next pings).
- Failure: if the location service restarts, the index refills within seconds from the next pings.

---

## 8. Case Study: Notification System

### Requirements
Send push, SMS, email, and in-app notifications reliably, respecting user preferences, at scale, with different priorities (OTP vs marketing).

### Design
```
Producers (services) -> Notification API -> validate, dedupe, prefs, rate limit
   -> priority queues per channel (Kafka/RabbitMQ)
   -> Channel workers (Push / SMS / Email) -> providers (FCM, APNs, Twilio, SES)
   -> delivery status callbacks -> status store / analytics
```

### Key decisions
- **Separate queues per channel and per priority:** a million-email campaign must never delay an OTP SMS.
- **User preferences and quiet hours:** a preferences service checked before sending; unsubscribe compliance.
- **Templates and localization:** template service with variables and language.
- **Rate limiting:** per user (don't spam), per provider (respect provider quotas).
- **Retries and DLQ:** exponential backoff on provider errors; fallback provider if one is down; poison messages to DLQ.
- **Idempotency:** `notificationId` plus dedupe store so retries don't duplicate.
- **Status tracking:** sent, delivered, opened/clicked via provider webhooks.
- **Scheduling:** delayed messages (TTL queues or a scheduler service).
- **Scale:** workers autoscale on queue depth; batch where providers support it.

---

## 9. Case Study: E-commerce Checkout and Inventory

### Requirements
Browse catalog, cart, place order, pay, ship. Must **never oversell** and must **never double charge**; survive flash sales.

### Services
Catalog, Search, Cart, Inventory, Order, Payment, Shipping, Notification; each with its own database.

### Read path (browsing)
Catalog DB is the source of truth; Elasticsearch for search; Redis and CDN for hot pages. Heavily cached and eventually consistent (a price change may take seconds to appear, but the price is **re-validated at checkout**).

### Write path (checkout) as a saga
```
1. Order Service: create order (PENDING)
2. Inventory: reserve stock (with TTL, e.g. 10 min)
3. Payment: authorize/charge (with idempotency key)
4. Order: mark CONFIRMED; Inventory: commit reservation
5. Events: Shipping, Notification, Analytics
Failures: payment fails -> release reservation, order CANCELLED
```
Use the **outbox pattern** for events and **orchestration** (a state machine) for clarity.

### Preventing overselling
- **Atomic conditional update:**
```sql
UPDATE inventory SET available = available - :qty
WHERE sku = :sku AND available >= :qty;   -- 1 row updated means reserved
```
- **Reserve then confirm** with TTL so abandoned carts return stock (a background sweeper or expiring keys).
- For extreme hot SKUs (flash sale): serialize through a **single-writer queue per SKU**, or pre-split stock into buckets (`sku#1..10`) to spread contention, or use Redis atomic `DECR` as a gate with periodic DB reconciliation.

### Flash sale protection
- **Virtual waiting room/queue** to admit users at a controlled rate.
- Aggressive caching of product pages (CDN), static fallback.
- Rate limits and bot protection (CAPTCHA, device checks).
- Pre-warmed capacity and autoscaling.
- Async order processing: accept the request into a queue and respond "processing", then confirm via notification.

### Payment correctness
- **Idempotency key** per checkout attempt.
- Payment states persisted (INIT, AUTHORIZED, CAPTURED, FAILED, REFUNDED); a **reconciliation job** compares with the PSP's records to fix timeouts and orphaned states.
- Card data never touches your servers (tokenization via PSP; PCI scope reduced).

---

## 10. Case Study: Collaborative Editor (Google Docs)

### Problem
Multiple users edit the same document simultaneously, seeing each other's changes in real time without conflicts.

### Approach
- **Transport:** WebSocket per client to a **document session server**; all users of a document are routed to the same server (consistent routing by `docId`) so ordering is simple.
- **Conflict resolution:**
  - **Operational Transformation (OT):** transform concurrent operations against each other so all replicas converge (Google Docs). The server serializes operations and transforms as needed.
  - **CRDTs:** data structures whose merges are commutative so replicas converge without a central transformer (Figma-like systems, Yjs, Automerge). Easier offline and peer-to-peer; more metadata overhead.
- **Persistence:** append operations to a log; periodic **snapshots** so loading doesn't replay millions of ops; version history built from snapshots/ops.
- **Presence and cursors:** ephemeral state over the same channel (not persisted).
- **Offline:** client queues ops; on reconnect, sync and transform/merge.
- **Scaling:** shard document sessions across servers by `docId`; hot documents (thousands of editors) are the hard case: cap active editors, or degrade presence updates.

---

## 11. Case Study: Web Crawler

### Requirements
Crawl billions of pages politely, avoid duplicates, keep content fresh.

### Design
```
Seed URLs -> URL Frontier -> Fetchers (DNS cache, robots.txt) -> Content store
          -> Parser -> extract links -> URL dedupe (Bloom filter) -> back to Frontier
          -> content dedupe (hash/SimHash) -> index pipeline
```
- **URL frontier:** prioritized queues (importance, freshness) plus **per-host queues** to enforce **politeness** (limit requests per domain, honor `robots.txt` and crawl-delay).
- **Distribution:** partition hosts across crawler nodes by `hash(host)` (so one node handles a host's politeness) with consistent hashing for membership changes.
- **Dedupe:** URL normalization plus Bloom filter for "seen"; content fingerprints (SimHash) for near-duplicate detection.
- **Traps:** limit depth/URL length/pages per domain (infinite calendars, session IDs in URLs).
- **Freshness:** re-crawl frequency based on change rate.
- **Failure handling:** retries with backoff, timeouts, blacklists for bad hosts.

---

## 12. Case Study: Payment System / Digital Wallet

### Requirements
Move money correctly. **Correctness over speed.** Auditability, no double spend, no lost money.

### Core design
- **Double-entry ledger:** every transaction creates balanced debit and credit entries in an **immutable, append-only** table. Balance is derived (or maintained and reconciled).
```sql
INSERT INTO ledger(txn_id, account_id, amount, direction) VALUES (...), (...);
-- sum(debits) = sum(credits) per txn_id (enforced in code and checks)
```
- **Strong consistency** for balances: relational DB with transactions; serializable or careful row locks / optimistic version checks on the account row.
- **Idempotency** keys on every request; unique constraints on `(idempotency_key)`.
- **State machine** per payment (CREATED, PENDING, SUCCEEDED, FAILED, REFUNDED) with legal transitions only.
- **External PSP/bank calls:** timeouts are ambiguous, so persist state *before* calling, handle "unknown" outcome with **status polling and webhooks**, and run **reconciliation** (daily) against PSP/bank statements.
- **Sagas/outbox** for cross-service steps (wallet debit, merchant credit, notification).
- **Fraud/risk checks** inline (rules, ML) with tight latency budgets; asynchronous deeper review.
- **Security/compliance:** PCI-DSS (tokenize cards, minimal scope), encryption, audit logs, strict access control, KYC/AML hooks.
- **Scaling:** shard by account/customer ID; keep each transaction within one shard when possible (transfers between shards use sagas with a holding/escrow account).

**Key principle:** it's acceptable to be *slow* or *temporarily unavailable*, but never *wrong*. Choose CP behavior.

---

## 13. Case Study: Distributed Cache (design a Redis-like system)

### Requirements
Low-latency key-value access, horizontal scale, high availability, eviction.

### Design
- **Partitioning:** consistent hashing with virtual nodes (or Redis Cluster's 16,384 hash slots); a client library or a proxy routes by key.
- **Replication:** each shard has a primary and 1-2 replicas; async replication for speed.
- **Failover:** failure detection via gossip/sentinel; promote the replica; clients refresh topology.
- **Eviction:** approximated LRU/LFU by sampling; per-key TTL via lazy expiration plus periodic sweeps.
- **Persistence (optional):** snapshots plus append-only log.
- **Hot keys:** client-side local caching, key replication, read from replicas.
- **Observability:** hit ratio, evictions, memory fragmentation, latency, connected clients.
- **Pitfalls:** big keys block the single-threaded engine (break up big values; avoid `KEYS *`), thundering herds (coalescing), and memory limits (set `maxmemory` plus a policy).

---

## 14. Case Study Template (Use for Any New Problem)

Copy this and fill it in:

```
1. Requirements
   Functional: ...
   Non-functional (scale, latency, availability, consistency): ...
   Out of scope: ...
2. Estimation
   DAU / QPS (avg, peak) / storage / bandwidth / cache size
3. API
   key endpoints, pagination, auth
4. Data model
   entities, access patterns, DB choice + why, shard key, indexes
5. High-level architecture
   write path / read path diagram
6. Deep dives (2-3)
   hardest components; alternatives and why chosen
7. Reliability
   SPOFs, replication, failure modes, retries/idempotency, degradation
8. Scale and evolution
   what breaks at 10x? what next?
9. Operations
   metrics, alerts, deploy strategy, security, cost
```

---

## 15. Practice Plan

**Week by week (45-60 min sessions):**
1. Week 1: URL shortener, pastebin, rate limiter (simple, teach caching/IDs/limits).
2. Week 2: Notification system, key-value store, web crawler (queues, partitioning, dedupe).
3. Week 3: Chat, news feed, search autocomplete (real-time, fan-out, tries).
4. Week 4: Video streaming, ride hailing, e-commerce checkout (media pipelines, geo, sagas).
5. Week 5: Payment system, distributed lock service, collaborative editor (correctness, consensus, CRDT/OT).
6. Ongoing: read engineering blogs (Netflix, Uber, Discord, Stripe, Cloudflare, Meta), and read *Designing Data-Intensive Applications*.

**Self-review checklist after each design:**
- Did I clarify requirements and estimate scale before drawing?
- Does every component trace to a requirement?
- Did I name the trade-offs and alternatives?
- Did I handle failures, duplicates, hot spots, and growth?
- Could I explain the read path and write path in 2 minutes each?

**Build to learn (for a Java/Spring Boot + React + MySQL + RabbitMQ + Docker stack):**
- Add Redis cache-aside to an existing endpoint and measure latency before and after.
- Add a RabbitMQ consumer with retry, DLQ, and an idempotency table.
- Implement the transactional outbox between two Spring Boot services.
- Put Resilience4j (timeout, retry, circuit breaker) on a service-to-service call, then kill the dependency and watch the behavior.
- Write a Redis token-bucket rate limiter and load test it with k6.
- Run the whole thing in Docker Compose, add Prometheus and Grafana, and build a RED dashboard.
