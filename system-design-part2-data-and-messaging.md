# System Design Deep Dive, Part 2: Data and Messaging

Pattern for each topic: **What / Why / How / Where / Example / Trade-offs**.

**Series:** Part 1: Foundations | Part 2 (this file) | Part 3: Distributed Systems and Reliability | Part 4: Interview Framework and Case Studies

---

## 1. Choosing a Database

### 1.1 Relational databases (SQL)

**What:** Data in tables with rows and columns, relationships through foreign keys, queried with SQL. Examples: MySQL, PostgreSQL, Oracle, SQL Server.

**Why:** Strong integrity (constraints, foreign keys), powerful queries (joins, aggregates), and **ACID** transactions. Decades of tooling.

**ACID explained with a bank transfer (A sends 100 to B):**
- **Atomicity:** debit A and credit B both happen or neither does.
- **Consistency:** rules hold (balance never negative, total money unchanged).
- **Isolation:** two concurrent transfers don't see each other's half-done state.
- **Durability:** once "committed", it survives a crash (write-ahead log on disk).

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1 AND balance >= 100;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

**Isolation levels (weakest to strongest):**

| Level | Prevents | Still allows |
|---|---|---|
| Read Uncommitted | nothing | dirty reads |
| Read Committed | dirty reads | non-repeatable reads |
| Repeatable Read (MySQL InnoDB default) | + non-repeatable reads | phantoms (mostly prevented in InnoDB via gap locks) |
| Serializable | everything | slowest, most blocking |

- **Dirty read:** you read data another transaction hasn't committed.
- **Non-repeatable read:** same row read twice gives different values.
- **Phantom read:** same query returns different *sets of rows*.

**Where:** Orders, payments, users, inventory, anything with relationships and correctness needs.
**Trade-offs:** Vertical scaling is easy, horizontal scaling (sharding) is hard; schema changes on huge tables need care.

### 1.2 NoSQL families

**What:** Non-relational stores optimized for specific access patterns and horizontal scale.
**Why:** When data volume, write throughput, or flexible schema outgrow a single relational node, or the access pattern is simple.

| Type | How it stores | Strength | Where | Example systems |
|---|---|---|---|---|
| **Key-value** | `key -> blob` | Extremely fast lookups | Sessions, carts, feature flags | Redis, DynamoDB |
| **Document** | JSON docs | Flexible schema, nested data | Product catalog, CMS, profiles | MongoDB |
| **Wide-column** | Rows with dynamic columns grouped by partition key | Huge write throughput, linear scale | Time-series, messaging, IoT, activity logs | Cassandra, HBase |
| **Graph** | Nodes and edges | Relationship traversal | Social graph, fraud rings, recommendations | Neo4j |
| **Search** | Inverted index | Full-text, relevance, aggregations | Product search, log analytics | Elasticsearch |
| **Time-series** | Timestamped points | Compression, downsampling | Metrics, monitoring | Prometheus, InfluxDB |
| **Object store** | Blobs in buckets | Cheap, durable, infinite | Images, video, backups | S3 |

**Example: modeling "user orders" in a document DB**
```json
{
  "_id": "order_981",
  "userId": 42,
  "items": [
    {"sku": "A1", "qty": 2, "price": 199},
    {"sku": "B7", "qty": 1, "price": 499}
  ],
  "status": "PAID"
}
```
One read returns the whole order (no joins). The cost: duplicating product data in each order, and harder cross-order analytics.

### 1.3 How to choose (decision guide)

Ask these in order:
1. **Do I need multi-row transactions and joins?** Use SQL.
2. **Is the access pattern just "get by key"?** Use key-value.
3. **Is write volume huge and append-heavy with predictable queries?** Use wide-column.
4. **Do I need full-text/relevance search?** Use Elasticsearch (as a *secondary* index, not the source of truth).
5. **Are relationships the main query ("friends of friends")?** Use graph.
6. **Big files?** Object storage.

**Polyglot persistence** (real systems): MySQL for orders (truth) + Redis for cache/sessions + Elasticsearch for search + S3 for images + Kafka for events.

---

## 2. Indexing

**What:** An extra data structure that lets the DB find rows without scanning the whole table (like a book's index).

**Why:** Without an index, `WHERE email = 'a@b.com'` scans millions of rows (O(n)). With one it's O(log n).

**How (B+ tree):**
- Balanced tree where each node is a disk page holding many keys (high fan-out, e.g. 100+).
- A 3-4 level tree can index hundreds of millions of rows; each lookup touches 3-4 pages.
- Leaf nodes are linked, so **range scans** are efficient.
- In InnoDB, the **clustered index** is the primary key: rows are physically stored in PK order. Secondary indexes store the PK, so a lookup needs a second jump (unless it's a covering index).

**Index types:**
- **B-tree:** equality and range. Default.
- **Hash:** equality only, O(1).
- **Composite:** multiple columns, order matters.
- **Covering:** includes all columns the query needs, so no table lookup.
- **Unique:** enforces uniqueness.
- **Full-text, spatial:** specialized.

**Example:**
```sql
CREATE INDEX idx_orders_user_created ON orders (user_id, created_at);

-- Uses the index (leftmost prefix: user_id, then range on created_at)
SELECT id, total FROM orders
WHERE user_id = 42 AND created_at >= '2026-01-01'
ORDER BY created_at DESC LIMIT 20;

-- Cannot use it efficiently (skips user_id, the leftmost column)
SELECT * FROM orders WHERE created_at >= '2026-01-01';
```

**Check with EXPLAIN:**
```sql
EXPLAIN SELECT * FROM orders WHERE user_id = 42;
-- Look for type=ref/range (good) vs type=ALL (full table scan, bad)
```

**Common mistakes:**
- `WHERE LOWER(email) = ...` or `WHERE DATE(created_at) = ...` wraps the column in a function, so the index is not used. Use a functional index or rewrite as a range.
- Leading wildcard `LIKE '%abc'` can't use a B-tree.
- Implicit type conversion (comparing a string column to a number).
- Indexing every column. Each index slows writes and uses storage.
- Low-cardinality columns (boolean) alone make poor indexes.

**LSM trees (the other major structure):**
- Writes go to an in-memory **memtable** plus a write-ahead log, then flush to immutable sorted **SSTables**; background **compaction** merges them.
- Writes are sequential, so they're very fast. Reads may check multiple files (mitigated by **Bloom filters**).
- Used in Cassandra, RocksDB, HBase, LevelDB.
- **B-tree vs LSM:** B-tree favors reads and in-place updates; LSM favors write-heavy workloads.

**Where:** Every database. Indexing is the first and cheapest performance fix.
**Trade-offs:** Faster reads vs slower writes, more storage, and planner confusion if you create too many.

---

## 3. Replication

**What:** Keeping copies of the same data on multiple nodes.

**Why:**
1. **Availability:** survive node failure.
2. **Read scaling:** serve reads from replicas.
3. **Latency:** place replicas near users.
4. **Backups / analytics** without hurting production.

### 3.1 Leader-follower (primary-replica)

**How:**
1. All writes go to the **leader**, which records them in a log (MySQL binlog, Postgres WAL).
2. Followers fetch and apply the log.
3. Reads can be served by followers.

**Sync vs async:**

| Mode | Behavior | Pros | Cons |
|---|---|---|---|
| Async | Leader acks immediately | Fast writes | Followers lag; a leader crash can lose recent writes |
| Sync | Leader waits for follower(s) | No data loss | Slower; blocked if the follower is down |
| Semi-sync | Wait for at least 1 follower | Balanced | Some added latency |

**Replication lag problems (very common in interviews):**
- **Read-your-writes violation:** User posts a comment, page refreshes (read from lagging replica), comment is missing.
  - Fix: for a short window after a write, read from the leader; or route by session; or track write timestamp/LSN.
- **Monotonic reads violation:** user sees data, refreshes, sees older data (hit a different replica).
  - Fix: pin a user to one replica.

**Failover:**
1. Detect leader failure (missed heartbeats).
2. Elect a new leader (the most up-to-date replica).
3. Reconfigure clients and old replicas.
- Risks: **split brain** (two leaders accepting writes), lost async writes, wrong failure detection. Use consensus-based tools (Orchestrator, Patroni, cloud-managed failover).

**Example (Spring Boot read/write split):**
```java
@Transactional(readOnly = true)   // route to replica via AbstractRoutingDataSource
public List<Order> history(long userId) { ... }

@Transactional                    // goes to primary
public Order place(OrderRequest r) { ... }
```

### 3.2 Multi-leader

**What:** Several nodes accept writes (often one per region).
**Why:** Low write latency in each region, tolerance for region outage.
**Cost:** **Write conflicts.** Two users edit the same record in different regions at the same time.
**Conflict resolution:** last-write-wins (simple, loses data), merge via CRDTs, application-level resolution, or avoid by routing each record to one home region.

### 3.3 Leaderless (Dynamo-style: Cassandra, DynamoDB, Riak)

**How:** The client or a coordinator sends each write to N replicas and each read to several.
**Quorum rule:** `R + W > N` guarantees a read overlaps the latest write.
- N=3, W=2, R=2: strong-ish consistency, tolerates 1 node down.
- N=3, W=1, R=1: fastest, eventual consistency.
- N=3, W=3, R=1: fast reads, writes fail if any node is down.
**Repair mechanisms:** read repair (fix stale replica on read), anti-entropy with Merkle trees, hinted handoff (store writes for a down node and replay).

**Where:** Replication underlies every production DB. Multi-leader for collaboration/multi-region; leaderless for high-availability write-heavy workloads.

---

## 4. Sharding (Partitioning)

**What:** Splitting a dataset across multiple machines so each holds a subset.

**Why:** Replication doesn't help when data doesn't fit on one node or writes exceed one node's capacity. Sharding scales **writes and storage** horizontally.

**When NOT to shard:** Exhaust cheaper options first: indexes, query tuning, caching, read replicas, bigger hardware, archiving old data. Sharding is operationally expensive.

### 4.1 Strategies

**Range-based:**
- `user_id 1-1M` on shard 1, `1M-2M` on shard 2.
- Pros: efficient range queries.
- Cons: **hot spots** (new users all hit the latest shard; time-based keys are worst).

**Hash-based:**
- `shard = hash(key) % N`.
- Pros: even distribution.
- Cons: range queries scatter across all shards; changing N remaps almost every key (massive data movement).

**Consistent hashing:**
- Hash both nodes and keys onto a ring (0 to 2^32). A key belongs to the first node clockwise.
- Adding/removing a node moves only about `1/N` of keys.
- **Virtual nodes:** each physical node gets many ring positions, giving even load and smoother rebalancing.
- Used by Cassandra, DynamoDB, memcached clients, CDNs.

```
Ring:  A(0)---k1---B(90)---k2---C(180)---k3---A'(270)
k1 -> B, k2 -> C, k3 -> A (wraps around)
Add node D at 130: only keys between B(90) and D(130) move from C to D
```

**Directory-based:** a lookup service maps key to shard. Flexible (move tenants individually), but the directory is a bottleneck/SPOF; cache it.

**Geo-based:** shard by region for latency and data-residency laws.

### 4.2 Choosing a shard key (the most important decision)

A good shard key has:
- **High cardinality** (many distinct values).
- **Even distribution** (no giant values).
- **Matches query patterns** (most queries hit one shard).
- **Stable** (rarely changes).

**Examples:**
- Multi-tenant SaaS: `tenant_id`. Great locality, but a huge tenant becomes a hot shard (give it a dedicated shard).
- Chat: `conversation_id`. All messages of a chat are on one shard.
- Social posts: `user_id`. Easy for "my posts", costly for "global feed".
- Bad: `country` (skewed), `created_date` (hot head), `is_active` (two values).

### 4.3 Problems you will face

| Problem | Explanation | Mitigation |
|---|---|---|
| Cross-shard joins | Data lives on different nodes | Denormalize, co-locate related data, application-level joins |
| Cross-shard transactions | 2PC is slow and fragile | Redesign to keep a transaction in one shard, or use sagas |
| Hot shard / celebrity | One key gets extreme traffic | Key salting (`userId#0..9`), dedicated shard, caching |
| Resharding | Moving data while serving traffic | Consistent hashing, over-provision logical shards (e.g. 1024 logical shards mapped to few machines) |
| Global secondary indexes | An index on a non-shard-key column | Local index (scatter-gather) or global index (extra writes, eventual) |
| Unique constraints | Uniqueness across shards | Include shard key, or central ID service |

**Pro tip:** Start with many *logical* shards (say 256) on a few physical machines. Growth then means moving logical shards between machines, not re-splitting data.

**Example (application-level routing in Java):**
```java
int shardId = (int) (Math.abs(userId.hashCode()) % LOGICAL_SHARDS);
DataSource ds = shardMap.get(shardId);   // logical shard -> physical DB
```

---

## 5. Denormalization and Data Modeling

**What:** Normalization removes duplication (each fact stored once). Denormalization intentionally duplicates data to make reads cheap.

**Why:** Joins across large or sharded tables are slow; read-heavy systems prefer pre-joined data.

**How:**
- Store `user_name` on each `post` row to avoid joining `users` for every feed item.
- Maintain counters (`likes_count`) on the parent row instead of `COUNT(*)` each time.
- Build **materialized views** for expensive aggregations, refreshed periodically or by events.

**Trade-off:** Updates must change many copies (e.g. user renames themselves). Use async propagation (events) and accept eventual consistency, or reference by ID and hydrate from cache.

**NoSQL rule:** model tables around **queries**, not entities. Example in Cassandra: want "messages in a chat, newest first", so:
```sql
CREATE TABLE messages_by_chat (
  chat_id uuid,
  sent_at timeuuid,
  sender_id uuid,
  body text,
  PRIMARY KEY ((chat_id), sent_at)
) WITH CLUSTERING ORDER BY (sent_at DESC);
```
`chat_id` is the partition key (which node), `sent_at` the clustering key (sort inside the partition).

---

## 6. OLTP vs OLAP, Warehouses and Lakes

- **OLTP:** many small read/write transactions (orders, logins). Row stores (MySQL, Postgres).
- **OLAP:** few huge analytical scans/aggregations ("revenue by region by month"). **Columnar** stores (BigQuery, Redshift, Snowflake, ClickHouse) read only needed columns and compress well.
- **ETL/ELT:** move data from OLTP into the warehouse. Don't run heavy analytics on your production DB.
- **Data lake:** cheap raw storage (S3 plus Parquet) for everything; **lakehouse** adds table semantics (Delta, Iceberg).
- **CDC (Change Data Capture)** streams DB changes (via binlog) to the warehouse, search index, and caches in near-real-time.

**Where:** The moment a manager asks for dashboards over 2 years of data, you need this separation.

---

## 7. Message Queues and Event Streams

### 7.1 What and why

**What:** A broker that sits between a producer (sender) and consumer (receiver) and holds messages.

**Why:**
1. **Decoupling:** producer doesn't need the consumer to be up or fast.
2. **Buffering spikes:** a Black Friday burst is absorbed and processed at a steady rate.
3. **Async work:** don't make the user wait for email, PDF generation, video encoding.
4. **Reliability:** retries, dead-letter queues.
5. **Fan-out:** one event, many consumers (billing, email, analytics).

**Example:** Checkout stores the order, publishes `OrderPlaced`, and returns in 80 ms. Inventory, email, loyalty points, and analytics each consume it independently. Email provider is down? Messages wait; nothing is lost.

### 7.2 Queue vs log

| | Traditional queue (RabbitMQ, SQS) | Distributed log (Kafka, Pulsar) |
|---|---|---|
| Model | Broker pushes messages; deleted after ack | Append-only log; consumers pull by offset |
| Replay | No (once acked, gone) | Yes (retention by time/size) |
| Ordering | Per queue | Per partition |
| Multiple independent consumers | Needs fan-out exchange or multiple queues | Natural via consumer groups |
| Throughput | Moderate to high | Very high (millions msgs/s per cluster) |
| Best for | Task queues, routing, work distribution | Event streaming, analytics, event sourcing, CDC |

### 7.3 RabbitMQ in detail

**Concepts:** Producer publishes to an **exchange**, which routes by **binding** rules to **queues**, from which consumers read.

| Exchange | Routing | Use |
|---|---|---|
| Direct | Exact routing key match | Task routing (`email`, `sms`) |
| Topic | Pattern (`order.*.created`) | Selective subscriptions |
| Fanout | All bound queues | Broadcast |
| Headers | Header attributes | Rare |

**Reliability toolkit:**
- **Durable queues + persistent messages** survive broker restart.
- **Publisher confirms:** broker confirms it stored the message.
- **Consumer acks:** message removed only after successful processing (manual ack).
- **Prefetch (QoS):** limit unacked messages per consumer for fair dispatch.
- **Dead letter exchange (DLX):** rejected/expired messages go to a DLQ for inspection.
- **Retry with delay:** TTL queue plus DLX back to the main queue.

**Example (Spring Boot + RabbitMQ):**
```java
@Configuration
class MqConfig {
  @Bean TopicExchange orders() { return new TopicExchange("orders"); }
  @Bean Queue emailQ() {
    return QueueBuilder.durable("email.q")
        .withArgument("x-dead-letter-exchange", "dlx")
        .build();
  }
  @Bean Binding b(Queue emailQ, TopicExchange orders) {
    return BindingBuilder.bind(emailQ).to(orders).with("order.placed");
  }
}

// Producer
rabbitTemplate.convertAndSend("orders", "order.placed", new OrderPlaced(orderId, userId));

// Consumer (idempotent!)
@RabbitListener(queues = "email.q")
public void onOrderPlaced(OrderPlaced e) {
    if (!processed.markIfNew(e.orderId())) return;   // dedupe
    emailService.sendReceipt(e);
}
```
```yaml
spring.rabbitmq.listener.simple.acknowledge-mode: auto   # ack on success, reject on exception
spring.rabbitmq.listener.simple.prefetch: 20
spring.rabbitmq.listener.simple.retry.enabled: true
```

### 7.4 Kafka in detail

**Concepts:**
- **Topic:** named stream, split into **partitions** (the unit of parallelism and ordering).
- **Producer** chooses partition by key hash: same key always goes to the same partition, so per-key ordering.
- **Consumer group:** each partition is read by exactly one consumer in the group. More consumers than partitions means idle consumers.
- **Offset:** a consumer's position; committed to Kafka. Replays by resetting offsets.
- **Replication:** each partition has a leader and followers (`replication.factor=3`, `min.insync.replicas=2`, producer `acks=all` for durability).
- **Retention:** keep for 7 days, or forever (compacted topics keep the latest value per key).

**Example:** Topic `orders` with 12 partitions, key = `userId`. All events for user 42 are ordered. A consumer group of 12 instances processes in parallel.

### 7.5 Delivery semantics

| Guarantee | Meaning | How it happens |
|---|---|---|
| At-most-once | May lose, never duplicates | Ack before processing |
| **At-least-once** | Never loses, may duplicate | Ack after processing; crash before ack causes redelivery |
| Exactly-once (effect) | Processed effectively once | Idempotent consumer, or Kafka transactions |

**Practical rule:** assume **at-least-once** and make consumers **idempotent**:
- Dedupe table with unique `message_id`.
- Natural idempotency (`SET status='PAID'`, upsert).
- Version checks.

### 7.6 Operational concerns

- **Poison messages:** a malformed message that always fails blocks the queue. Cap retries, then DLQ.
- **Backpressure:** consumers slower than producers cause queue growth. Monitor **queue depth and consumer lag**; autoscale consumers on lag; rate-limit producers.
- **Ordering:** parallel consumers break ordering. Partition by key if you need per-entity order.
- **Message size:** keep small; store large payloads in S3 and send a reference.
- **Schema evolution:** use Avro/Protobuf plus a schema registry; only add optional fields.

**Where queues appear:** email/SMS, image/video processing, order pipelines, webhooks delivery, audit logs, search index updates, ML feature pipelines.

---

## 8. Rate Limiting

**What:** Restricting how many requests a client can make in a time window.

**Why:** Prevent abuse and DDoS, ensure fair usage, protect downstream services, control cost (especially with paid APIs), and enforce plan tiers.

**How (algorithms):**

**Token bucket** (most popular)
- Bucket holds up to `capacity` tokens; refills at `rate` per second.
- Each request consumes 1 token; no token means reject (429).
- Allows **bursts** up to capacity, with a steady average rate.
- Example: capacity 20, rate 10/s means a burst of 20 instantly, then 10/s sustained.

**Leaky bucket:** requests queue and drain at a fixed rate; smooths output (traffic shaping), no bursts.

**Fixed window counter:** count per minute window. Simple, but a client can send 100 at 12:00:59 and 100 at 12:01:00, which is 200 in 2 seconds.

**Sliding window log:** store each request timestamp; exact but memory-heavy.

**Sliding window counter:** `current_count + previous_count x overlap%`. Good accuracy, low memory.

**Distributed implementation (Redis):**
```lua
-- token bucket in Lua (atomic)
local key = KEYS[1]
local rate, capacity, now = tonumber(ARGV[1]), tonumber(ARGV[2]), tonumber(ARGV[3])
local data = redis.call("HMGET", key, "tokens", "ts")
local tokens = tonumber(data[1]) or capacity
local ts = tonumber(data[2]) or now
tokens = math.min(capacity, tokens + (now - ts) * rate)
local allowed = tokens >= 1
if allowed then tokens = tokens - 1 end
redis.call("HMSET", key, "tokens", tokens, "ts", now)
redis.call("EXPIRE", key, 120)
return allowed and 1 or 0
```
Lua runs atomically in Redis, avoiding race conditions between "read count" and "write count".

**Response:** `429 Too Many Requests`, header `Retry-After: 30`, plus `X-RateLimit-Limit`, `X-RateLimit-Remaining`.

**Where to enforce:** API gateway (central), per-service middleware, and at the edge (CDN/WAF for IP-level DDoS).
**Key by:** user ID, API key, IP (weakest; NAT shares IPs), endpoint, or a combination.

**Design questions to settle:** fail-open or fail-closed if Redis is down (usually fail-open for general APIs, fail-closed for security-sensitive like login attempts). Multi-region: per-region limits or approximate global.

---

## 9. Unique ID Generation

**What:** Producing IDs for rows, messages, orders that are unique across the system.

**Why:** Auto-increment works on one DB, but with sharding, multi-region, and microservices you need IDs generated without a central bottleneck.

**Options:**

| Approach | Pros | Cons |
|---|---|---|
| DB auto-increment | Simple, sortable, compact | Single point; hard to shard; guessable; leaks volume |
| UUID v4 | No coordination | 128-bit, random, bad for B-tree locality (index fragmentation) |
| UUID v7 / ULID | Time-ordered plus random, index friendly | Slightly larger than 64-bit |
| **Snowflake** | 64-bit, time-sortable, no coordination | Needs unique machine IDs; clock-skew handling |
| Ticket server / ID ranges | Simple; each app grabs a block of 1000 | Central service; gaps on crash |

**Snowflake layout (64 bits):**
```
| 1 bit unused | 41 bits timestamp (ms) | 10 bits machine ID | 12 bits sequence |
```
- 41 bits of ms is about 69 years.
- 1024 machines, 4096 IDs per ms per machine (about 4M/s per machine).
- IDs are roughly time-sortable, so they work well as clustered keys and for "newest first" queries.
- Clock moving backward is a hazard: refuse to generate (or wait) until the clock catches up.

**Example (compact Java):**
```java
public synchronized long nextId() {
    long now = System.currentTimeMillis();
    if (now < lastTs) throw new IllegalStateException("clock moved backwards");
    if (now == lastTs) {
        seq = (seq + 1) & 0xFFF;
        if (seq == 0) while ((now = System.currentTimeMillis()) <= lastTs) {}
    } else seq = 0;
    lastTs = now;
    return ((now - EPOCH) << 22) | (machineId << 12) | seq;
}
```

**Where:** Tweets, orders, messages, any sharded table. For URL shorteners, base62-encode the ID to get short codes.

---

## 10. Full-Text Search (Elasticsearch)

**What:** A search engine built on an **inverted index**: word to list of documents containing it.

**Why:** SQL `LIKE '%shoe%'` can't use indexes and has no relevance ranking, typo tolerance, or stemming.

**How:**
1. **Analyze** text: lowercase, tokenize, remove stop words, stem ("running" to "run").
2. Build inverted index: `"shoe" -> [doc3, doc9, doc27]`.
3. Query: look up terms, intersect lists, **score** (TF-IDF / BM25), return top-K.
4. Distributed: index split into **shards**, each with **replicas**; query goes to all shards (scatter) and results are merged (gather).

**Where:** Product search, log search (ELK), autocomplete, analytics.
**Pattern:** keep MySQL as the **source of truth**; sync changes to Elasticsearch via CDC or events. Never treat the search index as your only copy.
**Trade-offs:** Eventual consistency (a new product isn't searchable for about a second), reindexing effort, memory hungry.

---

## 11. Object Storage and Large Files

**What:** Flat key-to-blob storage (S3, GCS, Azure Blob): cheap, durable (11 nines), practically unlimited.

**Why:** Databases are poor and expensive for images/videos. Object stores scale and serve through CDNs.

**How (upload pattern with pre-signed URLs):**
1. Client asks your API: "I want to upload `photo.jpg`".
2. API returns a **pre-signed URL** (time-limited, scoped).
3. Client uploads **directly to S3**, bypassing your servers.
4. S3 event notifies your backend (queue/Lambda) to process (thumbnails, virus scan).
5. DB stores only the object key and metadata.

**Large files:** **multipart/resumable uploads** (split into chunks; retry failed chunks only).
**Serving:** via CDN; private files via signed URLs with expiry.
**Where:** Profile pictures, documents, video, backups, logs, data lakes.

---

## 12. Part 2 Summary: Which Tool for Which Data Problem

| Problem | Reach for |
|---|---|
| Slow query | Index, `EXPLAIN`, rewrite query |
| Read load too high | Cache, then read replicas |
| Write load too high | Batch via queue, then shard |
| Data too big for one node | Shard (consistent hashing, logical shards) |
| Need to survive a DB node failure | Replication plus automated failover |
| Need free-text search | Elasticsearch fed from the primary DB |
| Spiky traffic / slow side-effects | Message queue |
| Replay events / many consumers | Kafka |
| Prevent abuse | Rate limiter (token bucket in Redis) |
| IDs across shards | Snowflake / UUIDv7 |
| Big media files | Object storage plus CDN plus pre-signed URLs |
| Heavy analytics | Warehouse (OLAP), fed by CDC |
