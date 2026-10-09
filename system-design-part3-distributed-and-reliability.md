# System Design Deep Dive, Part 3: Distributed Systems and Reliability

Pattern for each topic: **What / Why / How / Where / Example / Trade-offs**.

**Series:** Part 1: Foundations | Part 2: Data and Messaging | Part 3 (this file) | Part 4: Interview Framework and Case Studies

---

## 1. Why Distributed Systems Are Hard

**What:** A distributed system is multiple computers cooperating over a network to look like one system.

**Why it's hard:** Three things you can't escape:
1. **Partial failure:** some parts fail while others keep working, and you often can't tell which.
2. **Unreliable networks:** messages get lost, delayed, duplicated, reordered.
3. **Unreliable clocks:** machines disagree about the time.

**The core uncertainty:** You send a request and get no reply. Did the server never get it, process it but the reply was lost, or is it just slow? **You cannot know.** Every pattern in this part (timeouts, retries, idempotency, consensus) exists to cope with that.

**Fallacies of distributed computing:** the network is reliable; latency is zero; bandwidth is infinite; the network is secure; topology doesn't change; there's one administrator; transport cost is zero; the network is homogeneous.

**Example:** Payment service calls bank API, times out. If you retry blindly you might charge twice. If you don't, the customer might not be charged. The solution is an **idempotency key** (Section 7).

---

## 2. CAP Theorem and PACELC

### CAP

**What:** In the presence of a network **P**artition, a system must choose between:
- **C**onsistency: every read returns the most recent write (or an error).
- **A**vailability: every request to a non-failed node gets a (non-error) response.

**Why:** If the network splits nodes into two groups that can't talk, either you:
- **Refuse** some requests to avoid inconsistency (**CP**), or
- **Answer** from possibly stale/divergent data (**AP**).

**Important clarifications:**
- Partitions will happen, so "pick 2 of 3" is misleading. The real choice is C vs A **during a partition**.
- When no partition exists, you can have both.
- CAP's "consistency" means linearizability, not ACID's consistency.

**Example (two datacenters lose their link):**
- **CP system (e.g. etcd, ZooKeeper, HBase):** the minority side stops accepting writes (can't reach quorum). Users there see errors, but data stays correct. *Use for:* configuration, leader election, account balances.
- **AP system (e.g. Cassandra, DynamoDB):** both sides keep accepting writes; they reconcile later. Users never see errors but may see stale data or conflicts. *Use for:* shopping carts, likes, timelines, sensor data.

### PACELC

**What:** If **P**artition, choose **A** or **C**; **E**lse (normal operation), choose **L**atency or **C**onsistency.
**Why:** Even with no partition, strong consistency requires coordination between replicas, which costs latency. This explains why many databases offer tunable consistency.
**Example:** DynamoDB lets you choose eventually consistent reads (cheaper, faster) or strongly consistent reads (slower, 2x cost).

---

## 3. Consistency Models

**What:** Rules describing what values reads can return after writes in a replicated system.

**Why:** Stronger guarantees are easier to program against but cost latency and availability. Choose the weakest model that is still correct for the feature.

| Model | Guarantee | Example where it's enough |
|---|---|---|
| **Linearizable (strong)** | Behaves as if one copy; once a write completes, all later reads see it | Bank balance, inventory last item, leader lock |
| **Sequential** | Everyone sees operations in the same order (not tied to real time) | Some coordination services |
| **Causal** | If B depends on A, everyone sees A before B | Comments: a reply must not appear before its parent |
| **Read-your-writes** | You always see your own writes | Edit profile, then reload |
| **Monotonic reads** | You never go back in time | Don't show newer then older data |
| **Eventual** | Replicas converge if writes stop | Like counts, view counts, DNS |

**BASE** (the NoSQL counterpart to ACID): **B**asically **A**vailable, **S**oft state, **E**ventually consistent.

**Example (comment thread, causal):** Alice posts "Where's the party?". Bob replies "My place!". A third user must never see Bob's reply without Alice's post. Eventual consistency alone can show this anomaly; causal consistency prevents it.

**Decision per feature:**
- Money, stock, uniqueness (usernames): strong consistency.
- Social counters, feeds, recommendations: eventual is fine.
- User's own recent actions: read-your-writes.

---

## 4. Time, Clocks and Ordering

**What:** Determining the order of events across machines.

**Why:** Physical clocks drift (milliseconds to seconds even with NTP). "Last write wins by timestamp" can silently drop the *actual* latest write if clocks disagree.

**Tools:**
- **Lamport timestamps:** each node keeps a counter; on send, attach it; on receive, `counter = max(local, received) + 1`. Gives an order consistent with causality (if A caused B, `ts(A) < ts(B)`), but can't tell concurrent events apart.
- **Vector clocks:** each node tracks a counter per node. Can detect that two events are **concurrent** (a conflict) versus ordered. Used in Dynamo-style stores.
- **Hybrid Logical Clocks (HLC):** physical time plus logical counter; sortable and close to wall time (CockroachDB, YugabyteDB).
- **TrueTime (Google Spanner):** GPS and atomic clocks give a bounded uncertainty window; Spanner waits it out before committing to get external consistency.

**Example:** Two users edit a doc at 10:00:00.100 on server A and 10:00:00.090 on server B (B's clock is 50 ms slow, so B's edit was truly later). LWW picks A's edit and loses B's. Vector clocks would flag a conflict to resolve properly.

**Practical advice:** Never rely on wall-clock time across machines to order critical events. Use monotonically increasing sequence numbers from a single writer per entity (e.g. per-conversation sequence in chat) or a consensus-backed log.

---

## 5. Consensus: Raft and Leader Election

**What:** A protocol that lets a cluster of nodes agree on a value (or a sequence of values) even when some nodes crash or messages are lost.

**Why:** Needed for: electing a single leader, replicated state machines (config stores, metadata), distributed locks, consistent database commits. Without it you get **split brain** (two nodes both believe they're leader and corrupt data).

**Core idea:** A decision is final once a **majority (quorum)** accepted it. Any two majorities overlap, so two conflicting decisions can't both reach a majority.
- Cluster of `2f+1` nodes tolerates `f` failures: 3 nodes tolerate 1, 5 nodes tolerate 2.
- Use **odd** cluster sizes (4 nodes tolerate the same 1 failure as 3).

**How Raft works (simplified):**
1. **Roles:** Leader, Follower, Candidate. Time is divided into **terms**.
2. **Election:** followers expect heartbeats. If a follower hears nothing for a randomized timeout (e.g. 150-300 ms), it becomes a candidate, increments its term, and requests votes. A majority of votes wins. Randomized timeouts avoid repeated ties.
3. **Log replication:** clients talk to the leader. The leader appends a command to its log, sends it to followers, and when a **majority** has stored it, the entry is **committed** and applied to the state machine.
4. **Safety:** a candidate can only win if its log is at least as up-to-date as a majority's, so committed entries are never lost.

**Where:** etcd (Kubernetes' brain), Consul, Kafka KRaft (replaces ZooKeeper), CockroachDB, TiKV. ZooKeeper uses ZAB (similar). Paxos is the older, harder cousin (Spanner, Chubby).

**Example:** Kubernetes stores all cluster state in etcd. With a 3-node etcd cluster, losing one node keeps the cluster working; losing two means no majority, so writes halt (CP behavior).

**Trade-offs:** Every write needs a majority round trip, so throughput is limited and cross-region consensus is slow. Use consensus for **small, critical metadata**, not bulk data.

---

## 6. Distributed Transactions

### 6.1 The problem

**What:** An operation that must update data in multiple services/databases consistently. E.g. place order: create the order (Order DB), reserve stock (Inventory DB), charge the card (Payment DB).
**Why it's hard:** There's no single DB transaction spanning microservices with separate databases, and any step can fail midway.

### 6.2 Two-Phase Commit (2PC)

**How:**
1. **Prepare:** a coordinator asks each participant "can you commit?" Each does the work, locks resources, and votes yes/no, writing to its log.
2. **Commit/Abort:** if all voted yes, coordinator tells everyone to commit; otherwise abort.

**Problems:** Blocking: if the coordinator dies after prepare, participants hold locks indefinitely. High latency. Reduces availability. Poor fit for microservices and NoSQL.
**Where:** Inside some databases and XA transactions between a few trusted systems (e.g. JTA across a DB and JMS).

### 6.3 Saga pattern

**What:** A long transaction split into a sequence of **local transactions**, each with a **compensating transaction** that undoes it if a later step fails.

**Why:** Avoids distributed locks; each service keeps its own DB; scales and stays available. Cost: **eventual consistency** and no isolation (other requests can see intermediate states).

**Example: Order Saga**
```
1. Order Service:     create order (status = PENDING)      compensate: cancel order
2. Inventory Service: reserve stock                         compensate: release stock
3. Payment Service:   charge card                           compensate: refund
4. Order Service:     mark order CONFIRMED
If step 3 fails: release stock (2), cancel order (1).
```

**Two styles:**

| | Choreography | Orchestration |
|---|---|---|
| How | Services react to each other's events | A central orchestrator tells each service what to do |
| Pros | Loosely coupled, no central point | Clear flow, easier to reason about, monitor, and change |
| Cons | Hard to see the whole flow, cyclic dependencies | Orchestrator is extra logic (keep it as a state machine) |
| Tools | Kafka/RabbitMQ events | Temporal, Camunda, AWS Step Functions, or hand-rolled state machine |

**Design rules for sagas:**
- Compensations must be **idempotent** and must always be possible (design reversible steps; use "reserve then confirm", not "decrement forever").
- Persist saga state so recovery after crash is possible.
- Handle intermediate states in the UI ("Order processing...").
- Use **semantic locks**/status flags (`PENDING`) to prevent conflicting concurrent actions.

### 6.4 Transactional Outbox

**What:** A pattern to atomically update your DB **and** publish an event.
**Why:** The "dual write" problem. If you save to DB then publish to the broker, a crash between the two loses the event (or publishes an event for a rolled-back transaction). 

**How:**
1. In the **same local DB transaction**, write the business row and an `outbox` row containing the event.
2. A separate relay reads new outbox rows (polling, or **CDC** like Debezium on the binlog) and publishes to Kafka/RabbitMQ.
3. After publishing, mark the row as sent (or delete).
4. Consumers must be idempotent (relay can publish twice).

```java
@Transactional
public void placeOrder(OrderRequest r) {
    Order o = orderRepo.save(new Order(r));
    outboxRepo.save(new OutboxEvent(
        UUID.randomUUID(), "order.placed", toJson(new OrderPlaced(o.getId()))));
    // both rows commit atomically or neither does
}

@Scheduled(fixedDelay = 500)
public void relay() {
    for (OutboxEvent e : outboxRepo.findTop100ByPublishedFalseOrderById()) {
        rabbitTemplate.convertAndSend("orders", e.getType(), e.getPayload());
        e.markPublished();
    }
}
```

**Where:** Anywhere a service changes state and must reliably notify others. This is the standard fix for dual writes.

---

## 7. Idempotency

**What:** An operation is idempotent if doing it multiple times has the same effect as doing it once.

**Why:** Retries are unavoidable (timeouts, at-least-once queues, client double-clicks, network duplicates). Without idempotency, retries cause double charges, duplicate emails, duplicate orders.

**How:**

**1. Idempotency keys (for APIs):**
- Client generates a UUID per logical operation and sends `Idempotency-Key: <uuid>`.
- Server: on first sight, process and store `(key -> response)`; on repeat, return the stored response without re-processing.
- Use a unique constraint or atomic `SET NX` so two concurrent identical requests can't both pass.

```java
@PostMapping("/payments")
public ResponseEntity<PaymentResult> pay(@RequestHeader("Idempotency-Key") String key,
                                         @RequestBody PaymentRequest req) {
    return idempotencyStore.executeOnce(key, () -> paymentService.charge(req));
}
```

**2. Natural idempotency:** design operations as state assignments, not increments: `SET status = 'PAID'` not `balance = balance + 100` without a guard; use upserts (`INSERT ... ON DUPLICATE KEY UPDATE`).

**3. Deduplication in consumers:** table of processed `message_id`s with a unique constraint; TTL-clean old entries.

**4. Conditional updates / optimistic locking:**
```sql
UPDATE accounts SET balance = ?, version = version + 1
WHERE id = ? AND version = ?;   -- 0 rows updated means someone else changed it
```
```java
@Entity class Account { @Version long version; ... }   // JPA optimistic locking
```

**Where:** Payments, order creation, webhooks, queue consumers, saga steps.
**Trade-offs:** Needs storage and TTL for keys; define how long a key is valid (typically 24 h).

---

## 8. Distributed Locks and Coordination

**What:** Mutual exclusion across processes/machines.
**Why:** Prevent two workers from doing the same job (nightly billing run), or two requests from buying the last item.

**How:**
- **Redis:** `SET lock:job1 <random-token> NX PX 30000` acquires with a 30 s expiry; release only if the token still matches (Lua compare-and-delete). Fine for efficiency-level locking.
- **ZooKeeper / etcd:** ephemeral nodes or leases; stronger guarantees via consensus.
- **DB:** `SELECT ... FOR UPDATE`, or a row with a unique constraint.

**The big danger: a stale lock holder.**
1. Worker A takes lock (expires in 30 s).
2. A pauses (GC, network) for 40 s. The lock expires; B acquires it.
3. A resumes and still thinks it holds the lock; both write.

**Fix: fencing tokens.** The lock service hands out an increasing number with each acquisition (33, then 34). The storage layer rejects writes with a token lower than one it has already seen.

**Better: avoid locks.** Prefer optimistic concurrency (version columns), idempotent operations, single-writer partitioning (all events for entity X go to one consumer via Kafka key), or atomic DB statements.

**Example (flash-sale stock without a lock):**
```sql
UPDATE inventory SET qty = qty - 1 WHERE sku = 'X1' AND qty > 0;
-- affected rows = 1 means success; 0 means sold out. Atomic, no explicit lock.
```

**Where:** Leader election, scheduled-job singleton (ShedLock in Spring), resource allocation.

---

## 9. Failure Detection and Membership

**What:** Deciding which nodes are alive.
**Why:** You can't distinguish "dead" from "slow", so detection is probabilistic; wrong guesses cause unnecessary failovers or split brain.

**How:**
- **Heartbeats and timeouts:** simple; tune timeout against network jitter.
- **Phi-accrual detector:** outputs a suspicion level based on observed heartbeat distribution (Cassandra, Akka).
- **Gossip protocol:** every second each node shares its view with a few random peers. Information spreads exponentially (like a rumor), so it scales to thousands of nodes with no central coordinator (Cassandra, Consul/Serf, DynamoDB-style systems).

**Where:** Cluster membership, load balancer health checks, Kubernetes node/pod readiness.

---

## 10. Probabilistic Data Structures

These trade a small error for huge memory savings.

### Bloom filter
**What:** A bit array and k hash functions. Answers "is X in the set?" with **"definitely not"** or **"probably yes"**.
**How:** Add: set k bits. Query: if any of the k bits is 0, definitely absent. Never false negatives; tunable false-positive rate (about 1% at about 10 bits per element).
**Where:**
- LSM databases skip SSTables that definitely don't contain a key.
- Block cache-penetration attacks (check the filter before hitting the DB).
- "Has this URL been crawled?" or "username taken?" pre-check.
- Chrome's malicious URL check.
**Limit:** can't delete (use counting Bloom filter or cuckoo filter).

### HyperLogLog
**What:** Estimates count of **distinct** elements in about 12 KB with about 1% error.
**Where:** Unique visitors per page (Redis `PFADD` / `PFCOUNT`), distinct IPs, unique search queries.

### Count-Min Sketch
**What:** Approximate frequency of items with a small table.
**Where:** Trending hashtags, heavy hitters, abuse detection.

### Merkle tree
**What:** Hash tree where each parent hashes its children.
**Where:** Quickly detect which parts of two replicas differ (Cassandra anti-entropy), Git, blockchains.

---

## 11. Microservices

### 11.1 Monolith vs microservices

**What:** A monolith is one deployable. Microservices split the system into small, independently deployable services, each owning its data and one business capability.

**Why microservices:** Independent deploys and scaling per service, team autonomy, fault isolation, polyglot freedom.
**Why not (the hidden costs):** network latency and failures, distributed transactions, observability and debugging, versioning/contracts, operational overhead, data consistency challenges.

| | Monolith | Microservices |
|---|---|---|
| Deploy | All together | Per service |
| Scale | Whole app | Per service |
| Data | Shared DB | DB per service |
| Complexity | Inside code | In operations and network |
| Best for | Startups, small teams, unclear domain | Large orgs, clear boundaries, independent scaling |

**Recommended path:** Start with a **modular monolith** (clear packages/modules, enforced boundaries). Extract services when there's a real driver: a team bottleneck, one component with very different scaling, different release cadence.

**Anti-pattern:** the **distributed monolith**, where services are tightly coupled, share a DB, or must deploy together. You get all the costs and none of the benefits.

### 11.2 How to split: bounded contexts (DDD)

Split along **business capabilities**, not technical layers: Order, Inventory, Payment, Catalog, Notification, User. Each service owns its data and exposes an API/events. Other services never read its tables.

### 11.3 Communication

| Style | Use when | Risk |
|---|---|---|
| **Sync (REST/gRPC)** | Need an immediate answer (price check) | Latency chains, cascading failures, availability multiplies down |
| **Async (events/queues)** | Fire-and-forget, fan-out, eventual consistency OK | Harder to trace, ordering/duplicates |

**Rule of thumb:** Commands/queries that need an answer now are sync; notifications of "something happened" are async events.

### 11.4 Supporting pieces

- **Service discovery:** services find each other by name (Eureka, Consul, Kubernetes DNS) rather than hardcoded IPs.
- **Centralized config:** Spring Cloud Config, Consul, K8s ConfigMaps/Secrets.
- **API gateway** at the edge; **BFF** (Backend-for-Frontend) per client type (web, mobile).
- **Service mesh (Istio, Linkerd):** sidecar proxies handle mTLS, retries, timeouts, telemetry without app code.
- **Strangler fig migration:** put a facade in front of the monolith; gradually route features to new services until the monolith is hollowed out.

---

## 12. Resilience Patterns

**Why:** In a system of many services, something is always failing. The goal is to keep failures **contained** and degrade gracefully.

### 12.1 Timeouts

**What:** Every remote call has a maximum wait.
**Why:** Without one, a slow dependency makes threads pile up until your service dies too (**cascading failure**).
**How:** Set per dependency from its p99 latency plus margin; propagate **deadlines** (remaining time budget) downstream so you don't work for a client that has already given up.

### 12.2 Retries with exponential backoff and jitter

**What:** Re-attempt failed calls with growing, randomized delays.
**Why:** Many failures are transient (network blip, brief overload).
**How:** delay = `min(cap, base * 2^attempt)` randomized (jitter).
**Rules:** retry only **idempotent** operations and only on retryable errors (timeouts, 503; not 400/404). Cap attempts. Without jitter, thousands of clients retry in sync (**retry storm**) and crush a recovering service. Beware **retry amplification**: 3 layers each retrying 3 times means 27 calls.

### 12.3 Circuit breaker

**What:** A wrapper that stops calling a failing dependency.
**States:**
- **Closed:** normal; counts failures.
- **Open:** failure rate exceeded the threshold, so calls fail immediately (no waiting) for a cool-down period.
- **Half-open:** allow a few probe calls; success closes it, failure re-opens it.
**Why:** Fail fast, protect the failing service from more load, free your threads.

**Example (Resilience4j in Spring Boot):**
```java
@CircuitBreaker(name = "inventory", fallbackMethod = "stockFallback")
@Retry(name = "inventory")
@TimeLimiter(name = "inventory")
public CompletableFuture<Stock> getStock(String sku) {
    return CompletableFuture.supplyAsync(() -> inventoryClient.stock(sku));
}
public CompletableFuture<Stock> stockFallback(String sku, Throwable t) {
    return CompletableFuture.completedFuture(Stock.unknown(sku)); // degrade gracefully
}
```
```yaml
resilience4j.circuitbreaker.instances.inventory:
  slidingWindowSize: 20
  failureRateThreshold: 50
  waitDurationInOpenState: 15s
  permittedNumberOfCallsInHalfOpenState: 3
```

### 12.4 Bulkhead

**What:** Isolate resources (thread pools, connection pools, semaphores) per dependency, like watertight compartments in a ship.
**Why:** If the recommendations service hangs, it exhausts only *its* pool, not the threads needed for checkout.

### 12.5 Fallbacks and graceful degradation

**What:** A reduced-functionality alternative when something fails.
**Examples:** Show cached/popular items if personalization is down; hide the "recommended for you" widget; accept the order and confirm payment later; serve stale cache.

### 12.6 Load shedding and backpressure

- **Load shedding:** under overload, deliberately reject low-priority requests (return 503) to keep serving critical ones, rather than degrading for everyone.
- **Backpressure:** signal producers to slow (bounded queues, reactive streams, 429s).
- **Rate limits and queue length limits:** never use unbounded queues.

### 12.7 Health checks

- **Liveness:** "am I stuck? restart me" (should be cheap, not check dependencies).
- **Readiness:** "can I serve traffic now?" (dependencies OK, warmed up).
- Kubernetes uses both; Spring Boot Actuator exposes `/actuator/health/liveness` and `/readiness`.

### 12.8 Redundancy and chaos testing

- Eliminate single points of failure at every layer: multiple instances, multi-AZ, replicated DBs, redundant LBs.
- **Chaos engineering:** deliberately kill instances, add latency, and cut dependencies in controlled experiments (Chaos Monkey, Litmus, Gremlin) to prove resilience before real outages do.

---

## 13. Event-Driven Architecture, CQRS, Event Sourcing

### 13.1 Event-driven

**What:** Services communicate by publishing **events** (facts: `OrderPlaced`) that others subscribe to.
**Why:** Loose coupling, easy to add new consumers, natural async scaling.
**Styles:**
- **Event notification:** thin event (`orderId` only); consumers call back for details.
- **Event-carried state transfer:** event includes the data, so consumers keep local copies (fewer calls, more duplication).
**Pitfalls:** hard to trace flows, eventual consistency, schema evolution, duplicate/out-of-order events.

### 13.2 CQRS (Command Query Responsibility Segregation)

**What:** Separate the **write model** (commands) from the **read model** (queries).
**Why:** Reads and writes often have different shapes and scale. Writes need validation and normalized transactional storage; reads need denormalized, fast, query-specific views.
**How:** Commands update the primary store and emit events; an event handler updates one or more read stores (Redis, Elasticsearch, a denormalized SQL table).
**Example:** Order writes go to MySQL (normalized). A projector builds `order_summary` documents in Elasticsearch for the "My Orders" page and search.
**Cost:** eventual consistency between sides, more moving parts. Don't use CQRS for simple CRUD.

### 13.3 Event Sourcing

**What:** Instead of storing current state, store the **sequence of events** that led to it; current state = replay of events (plus periodic snapshots).
**Why:** Perfect audit trail, time travel, rebuild any view, debugging ("how did the balance become X?").
**Example:** Account events: `Opened(0)`, `Deposited(500)`, `Withdrawn(200)`. Balance = 300 by folding events.
**Cost:** Event schema evolution, replay time (use snapshots), querying needs projections (hence usually paired with CQRS), steep learning curve.
**Where:** Finance/ledgers, order lifecycle, audit-critical domains.

---

## 14. Deployment and Infrastructure

### 14.1 Containers and Docker

**What:** A container packages your app and its dependencies into an isolated, portable unit sharing the host OS kernel.
**Why:** "Works on my machine" disappears; fast startup; efficient density; consistent from laptop to production.
**Example:**
```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-XX:MaxRAMPercentage=75","-jar","app.jar"]
```

### 14.2 Kubernetes

**What:** A container orchestrator.
**Why / what it gives you:** scheduling, **self-healing** (restarts failed pods), **autoscaling** (HPA), **rolling updates** and rollbacks, service discovery and load balancing, config/secret injection.
**Key objects:** Pod (smallest unit), Deployment (desired replicas and rollout), Service (stable address and LB), Ingress (HTTP routing), ConfigMap/Secret, StatefulSet (stable identity, for DBs/brokers), HPA, PersistentVolume.

### 14.3 Deployment strategies

| Strategy | How | Pros | Cons |
|---|---|---|---|
| **Rolling** | Replace instances gradually | No extra capacity needed | Two versions live at once; slow rollback |
| **Blue-green** | Two identical environments; switch traffic | Instant switch and rollback | Double infra cost; DB changes need care |
| **Canary** | Send 1% to 5% to the new version, watch metrics, then expand | Limits blast radius | Needs good metrics and routing |
| **Feature flags** | Ship code dark, enable per user/percentage | Decouples deploy from release; kill switch | Flag debt |

**DB migrations with zero downtime (expand, migrate, contract):**
1. **Expand:** add the new column/table (backward compatible).
2. Deploy code that writes both and reads new, backfill data.
3. **Contract:** remove the old column in a later release.
Never ship code and a breaking schema change in the same step.

### 14.4 Autoscaling

Scale by CPU/memory, request rate, **queue depth/consumer lag** (best for workers), or custom metrics. Use cool-downs to avoid flapping; pre-scale before known peaks; remember DB connection limits grow with instance count.

---

## 15. Observability

**What:** Being able to understand a system's internal state from its outputs.
**Why:** In distributed systems you can't attach a debugger. You need data to answer "what's broken, where, and why?".

### 15.1 The three pillars

**Metrics:** numeric time series (Prometheus plus Grafana, CloudWatch).
- **RED** for services: **R**ate, **E**rrors, **D**uration.
- **USE** for resources: **U**tilization, **S**aturation, **E**rrors.
- **Four golden signals (Google SRE):** latency, traffic, errors, saturation.
- Use **histograms** for latency to compute p95/p99.

**Logs:** structured JSON with a **correlation/trace ID** in every line, centralized (ELK/EFK, Loki). Don't log secrets/PII; use levels sensibly.

**Traces:** one request's path across services as spans (OpenTelemetry, Jaeger, Zipkin). Shows which hop is slow.

### 15.2 Alerting

- Alert on **symptoms** (users affected: error rate, latency SLO burn), not every cause (CPU 80%).
- Every alert should be **actionable** and link a runbook.
- Fatigue kills on-call: tune or delete noisy alerts.
- **Blameless postmortems** after incidents; track action items.

**Example (Spring Boot):** Actuator plus Micrometer expose `/actuator/prometheus`; Prometheus scrapes; Grafana dashboards show RED; an alert fires if `5xx rate > 2%` for 5 minutes.

---

## 16. Security Essentials

**Principles:** least privilege, defense in depth, secure by default, never trust input, encrypt in transit and at rest.

### 16.1 Authentication and authorization

- **AuthN** = who are you. **AuthZ** = what may you do.
- **Sessions (server-side):** session ID cookie; server stores state; easy revocation; needs shared store when scaled.
- **JWT (token-based):** signed token carrying claims; stateless validation; hard to revoke, so use **short-lived access tokens (about 15 min) plus refresh tokens** (rotate and store server-side to allow revocation).
- **OAuth 2.0:** delegated *authorization* ("let this app read my calendar"). **OIDC** adds *authentication* (identity) on top. Use **Authorization Code flow with PKCE** for SPAs and mobile.
- **RBAC** (roles: admin, editor) vs **ABAC** (attributes/policies).

### 16.2 Common attacks and defenses

| Attack | Defense |
|---|---|
| SQL injection | Parameterized queries / ORM; never concatenate input |
| XSS | Output encoding, CSP, sanitize HTML, `HttpOnly` cookies |
| CSRF | SameSite cookies, CSRF tokens |
| Broken access control (IDOR) | Check ownership on every request, not just login |
| SSRF | Allow-list outbound targets; block internal IP ranges |
| Credential stuffing / brute force | Rate limiting, MFA, lockouts, breached-password checks |
| DDoS | CDN/WAF, rate limiting, autoscaling, anycast |
| Secrets leakage | Vault / secrets manager; never commit; rotate |

**Passwords:** hash with bcrypt/scrypt/Argon2 (slow, salted). Never MD5/SHA-1/plain.
**Transport:** TLS everywhere; **mTLS** between services; HSTS.
**Data:** encrypt at rest; field-level encryption for sensitive data; tokenization for cards (PCI-DSS); audit logs.

**Example (parameterized query, Java):**
```java
// Safe
jdbc.query("SELECT * FROM users WHERE email = ?", mapper, email);
// Vulnerable
jdbc.query("SELECT * FROM users WHERE email = '" + email + "'", mapper);
```

---

## 17. Disaster Recovery and High Availability

**What:** Plans and architecture to survive and recover from major failures (AZ, region, data corruption, human error).

**Key terms:**
- **RPO (Recovery Point Objective):** maximum acceptable **data loss** (time). RPO = 5 min means you can lose the last 5 min of data.
- **RTO (Recovery Time Objective):** maximum acceptable **downtime**.

**Strategies (cheapest and slowest to costliest and fastest):**

| Strategy | RTO/RPO | How |
|---|---|---|
| Backup and restore | Hours / hours | Periodic backups to another region |
| Pilot light | Tens of minutes | Minimal core (DB replica) always on; scale up on disaster |
| Warm standby | Minutes | Scaled-down full copy running |
| Active-active multi-region | Near zero | Full capacity in 2+ regions, traffic split |

**Practices:**
- **Multi-AZ** for routine hardware/datacenter failure; **multi-region** for regional outages and global latency.
- **Backups:** automated, encrypted, in another region/account, with **tested restores** (an untested backup is a hope, not a backup). Point-in-time recovery for DBs.
- **Protect against human error:** soft deletes, deletion protection, immutable backups, staged rollouts.
- Run **game days** to rehearse failover.

---

## 18. Multi-Region Design

**What:** Running a system across geographic regions.
**Why:** Lower latency for global users, survive regional outages, data residency laws (GDPR, India's DPDP).

**Patterns:**
- **Active-passive:** one region serves; another is standby. Simple; failover takes minutes; standby capacity is mostly idle.
- **Active-active:** all serve traffic. Best latency and availability; hard part is **data** (conflicts, replication lag).
- **Geo-partitioning (home region per user):** each user's data lives in their region; most requests are local; cross-region is rare. Often the most practical.

**Traffic routing:** GeoDNS, latency-based DNS, anycast, global load balancers (health-check-driven).
**Data:**
- Single-writer region per dataset plus async replicas elsewhere (reads local, writes may cross regions).
- Multi-leader with conflict resolution/CRDTs.
- Globally consistent DBs (Spanner, CockroachDB): correct but cross-region write latency (tens to hundreds of ms).

**Gotchas:** speed of light (about 70-150 ms between continents), replication lag during failover (RPO > 0 for async), cross-region data transfer cost, compliance, testing failover.

---

## 19. Performance Engineering

**Workflow:** measure first, find the **bottleneck**, fix that, measure again. Don't guess.

**Toolkit:**
- **Profile** (flame graphs, async-profiler, `EXPLAIN`).
- **Load test** (k6, JMeter, Gatling) to find the breaking point.
- Common fixes: add index, fix **N+1 queries**, cache, batch operations, connection pooling, compression, pagination, async I/O, reduce payload size, CDN.

**N+1 example:**
```java
// N+1: 1 query for orders + N queries for items
for (Order o : orderRepo.findAll()) { o.getItems().size(); }
// Fix: fetch join / entity graph
@Query("select o from Order o join fetch o.items")
List<Order> findAllWithItems();
```

**Useful laws:**
- **Little's Law:** `L = lambda x W` (concurrent requests = arrival rate x time in system). If you handle 500 req/s and each takes 200 ms, you have about 100 requests in flight, which sizes thread/connection pools.
- **Queueing intuition:** latency grows sharply as utilization nears 100%. Keep headroom (about 60-70% target utilization).
- **Tail latency amplification:** a request that fans out to 100 backends is slow if *any* is slow; mitigate with timeouts, hedged requests, reducing fan-out.
- **Amdahl's law:** the serial fraction limits speedup from parallelism.

---

## 20. Part 3 Summary: Failure to Pattern Map

| Failure / need | Pattern |
|---|---|
| Dependency is slow | Timeout, circuit breaker, bulkhead |
| Transient errors | Retry with backoff and jitter (idempotent only) |
| Duplicate requests/messages | Idempotency keys, dedupe table |
| Update DB and send event atomically | Transactional outbox plus CDC |
| Multi-service business transaction | Saga (orchestrated or choreographed) |
| Two nodes think they're leader | Consensus (Raft), fencing tokens |
| Concurrent update of same row | Optimistic locking (version), atomic SQL |
| Overload | Rate limit, load shedding, backpressure, autoscale |
| Stale replicas confuse users | Read-your-writes routing |
| Need audit/history | Event sourcing |
| Read/write shapes diverge | CQRS |
| Region outage | Multi-region, tested failover |
| "What's wrong?" | Metrics (RED), logs with trace IDs, distributed tracing |
