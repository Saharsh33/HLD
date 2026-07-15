# System Design Glossary — Every Term You'll Ever Need

**What this file is:** one place to quickly look up any tech term that shows up in system design interviews. No fluff, no textbook definitions — explained like you're telling a friend over coffee, but precise enough to say word-for-word to an interviewer.

**How to use it:** Ctrl+F the term. Read the 2-3 line explanation. If you want to go deeper, the "where it shows up" tells you which system design note dives into it.

---

## Foundational Concepts

### CAP Theorem
In a distributed system, when the network between servers breaks (a **partition**), you have to pick one: keep serving requests with possibly-stale data (**Availability**) or refuse to answer until all servers agree (**Consistency**). You literally cannot have both during the outage — that's the theorem.

- **AP (Availability + Partition tolerance):** Instagram, WhatsApp, Netflix, YouTube, Tinder — a stale feed or late read-receipt is harmless.
- **CP (Consistency + Partition tolerance):** Stripe, BookMyShow (during checkout) — a stale bank balance or double-sold seat is a real problem.
- Some systems switch mid-request: Uber uses AP for the GPS dot on the map but CP for assigning a driver to a ride.

### ACID
The four promises a relational database makes about a transaction:
- **Atomicity:** debit sender AND credit receiver — both happen or neither happens. No half-finished state.
- **Consistency:** the DB never violates its own rules (balance can't go negative if you said so).
- **Isolation:** two transactions hitting the same row at the same millisecond can't corrupt each other — the end result looks like they ran one after another.
- **Durability:** once the DB says "committed," that data survives a crash, power outage, whatever.

If an interviewer asks "when do you need ACID?" → whenever a wrong intermediate state causes real-world damage (money, double-booking, permission escalation).

### Eventual Consistency
Data becomes correct everywhere *eventually* (usually within seconds), just not at the exact millisecond it was written. If User A posts a photo and User B sees it 2 seconds later — that's eventual consistency in action. Perfectly fine for feeds, terrible for bank balances.

### Strong Eventual Consistency
Stricter than regular eventual consistency. Guarantees that if two replicas have received the **same set of operations** (in any order), they show the **identical** state. Regular eventual consistency allows temporary divergence — strong eventual consistency doesn't. Google Docs needs this; Instagram doesn't.

### BASE
The NoSQL counterpart to ACID — **B**asically **A**vailable, **S**oft state, **E**ventual consistency. It's a design philosophy, not a protocol: prioritize availability and accept that data will be temporarily inconsistent. Think of it as the opposite stance from ACID.

### Horizontal Scaling vs Vertical Scaling
- **Vertical scaling (scale up):** add more CPU/RAM/SSD to a single machine. Simple, but there's a ceiling — you can't buy a server with 10 TB of RAM.
- **Horizontal scaling (scale out):** add more machines and distribute the load. No ceiling, but now you need sharding, load balancing, and distributed coordination. Every system in your notes scales horizontally — that's the whole point of distributed design.

### Latency vs Throughput
- **Latency:** how long one request takes (measured in ms). A user cares about this.
- **Throughput:** how many requests the system handles per second (measured in QPS/RPS). The business cares about this.

They're often at tension: optimizing for one can hurt the other. Batching increases throughput but adds latency. Caching reduces latency but doesn't directly increase write throughput.

### P99 / P95 Latency (Percentile Latency)
P99 = the latency that 99% of requests are faster than. If your P99 is 200ms, it means 99 out of 100 requests complete in under 200ms — but 1 in 100 might take longer. Averages are useless for latency because they hide the slow requests. Interviewers expect you to say "P99 < 200ms" not "average < 200ms."

### SLA / SLO / SLI
- **SLA (Service Level Agreement):** a legal contract with users — "99.99% uptime or we refund you." The business commitment.
- **SLO (Service Level Objective):** an internal target — "we aim for 99.99% uptime." Stricter than the SLA so you have buffer.
- **SLI (Service Level Indicator):** the actual metric being measured — "uptime was 99.97% last month." The data that tells you if you're meeting the SLO.

In an interview, when you say "99.99% availability" — that's an SLO. If you say "we'd put this in the SLA" — that's the contractual version.

---

## Databases & Storage

### PostgreSQL (Postgres)
The go-to relational database when you need ACID, complex joins, referential integrity, or strong consistency. Handles structured data like user profiles, permissions, social graphs, payment ledgers. Struggles at millions of writes/sec or hundreds of TB — that's when you reach for something else.

### Cassandra / ScyllaDB
Wide-column NoSQL database built for insane write throughput and append-only time-series data. Uses an **LSM-tree** storage engine — writes go to memory first, then flush to disk sequentially. No row-level locking, no joins, no ad-hoc queries. Partition by something like `user_id`, cluster by `timestamp`, and "give me the last 50 messages" is a fast sequential disk read.

ScyllaDB is a C++ rewrite of Cassandra — same data model, better performance per node.

### Redis
In-memory key-value store. Everything lives in RAM, so reads/writes are sub-millisecond. Used for caching, session storage, distributed locks, rate limit counters, pub/sub messaging, and geospatial queries. Single-threaded execution = no race conditions between commands.

Key Redis features you should know:
- **SETNX (SET if Not eXists):** only the first caller succeeds, everyone else gets rejected. The backbone of distributed locking.
- **GEOADD / GEORADIUS:** store GPS coordinates and query "find everything within 3 km of this point" — all in RAM, < 1 ms.
- **ZSET (Sorted Set):** every item has a score (e.g., timestamp). Redis keeps them automatically sorted. "Give me the newest 20" is instant.
- **INCR / DECR:** atomic counter operations. 10,000 simultaneous requests incrementing the same counter will never lose a count.
- **Pub/Sub:** publish a message to a channel, all subscribers get it instantly. Used for real-time fan-out (chat, live seat maps, notifications).
- **Lua scripts:** execute multiple Redis commands atomically — the entire script runs as one unit, no interleaving.

### Elasticsearch
Full-text search engine built on inverted indices. When you need fuzzy search, autocomplete, faceted filtering (by genre, year, language), or multi-field search across millions of records at high QPS — Postgres `LIKE '%query%'` won't cut it, Elasticsearch will.

### S3 / GCS (Object Storage)
Cheap, infinitely scalable blob storage for files — images, videos, PDFs, backups. 11 nines of durability (99.999999999%). Never store binary files in a database — it bloats indexes, kills buffer pools, and makes backups take forever. Store the file in S3, store the URL in the database.

### WORM Storage (Write Once, Read Many)
Storage where once data is written with a retention lock, nobody — not even the root admin — can delete or modify it until the timer expires. Used for regulatory compliance (financial audit logs, healthcare records). S3 Object Lock in compliance mode is the AWS implementation.

### B-Tree
The data structure behind indexes in relational databases (Postgres, MySQL). A balanced tree that keeps data sorted on disk, enabling O(log N) lookups, range scans, and ordered reads. Updates require modifying the tree in place (random disk writes) — which is why B-Trees are slower for write-heavy workloads than LSM-Trees.

### LSM-Tree (Log-Structured Merge Tree)
The storage engine behind Cassandra, ScyllaDB, RocksDB, and LevelDB. Writes go to an in-memory buffer first (fast), then flush to disk as sorted immutable files. Reads merge data across multiple files. The key advantage: writes are sequential (append-only), not random — making them much faster than B-Tree updates. The trade-off: reads can be slower because they may need to check multiple files.

### R-Tree
A spatial index structure used by PostGIS. Organizes geographic data into nested bounding rectangles, enabling fast "find everything within this area" queries. Think of it as a B-Tree but for 2D space instead of 1D values.

### WAL (Write-Ahead Log)
A technique where every database change is first written to an append-only log file *before* being applied to the actual data. If the server crashes mid-write, the WAL lets it recover by replaying the log. Every serious database (Postgres, MySQL, Cassandra) uses a WAL. Debezium/CDC tools read this log to stream changes externally.

### Read Replicas
Read-only copies of a database that stay in sync with the primary (leader) via replication. Route all reads to replicas, writes to the primary. This is how you scale a read-heavy system (100:1 read-write ratio like Instagram) without adding write complexity. Trade-off: replicas can be slightly behind the primary (replication lag).

### Connection Pooling
Instead of opening a new database connection for every request (expensive — TCP handshake + TLS + auth each time), maintain a pool of pre-opened connections and reuse them. Tools like PgBouncer (Postgres) or HikariCP (Java) manage this. Without pooling, a traffic spike exhausts the database's max connection limit instantly.

---

## Caching & Performance

### CDN (Content Delivery Network)
A global network of edge servers that cache your static content (images, videos, JS/CSS) close to users. A viewer in Tokyo pulls the video from a Tokyo CDN server, not your origin server in Virginia. Massively reduces latency and origin server load.

**Origin Pull:** when the CDN doesn't have the file cached yet, it fetches it once from your real storage (the "origin"), caches it, and every subsequent nearby request gets the cached copy.

### Cache Stampede (Thundering Herd on Read)
A popular cache key expires. 50,000 concurrent requests all see a cache miss at the same instant and all slam the database for the exact same data. Database connection pool exhausts, cascading failures follow.

**Fix:** distributed mutex — use Redis `SETNX` so only one thread rebuilds the cache. Everyone else waits 50ms and retries the cache, finding freshly populated data.

### Hot Key / Hot Partition
One database row, cache key, or partition gets so much traffic it overloads its single node while the rest of the cluster sits idle. Like one supermarket checkout lane with 5,000 people while 99 other lanes are empty.

**Fixes vary:** balance slotting (split one row into N rows), key salting (append a random bucket number), local aggregation (batch updates before flushing to the central store).

### Write Amplification
One user action triggering a disproportionate number of hidden backend writes. A celebrity with 600M followers posts once → the system tries to write to 600M timeline caches. The "amplification" is the ratio of downstream writes to the original action.

### Fan-Out
Taking one piece of data and distributing it to many recipients. **Fan-out-on-write (push model):** when a user posts, immediately push the post to all followers' feeds. Fast reads, expensive writes. **Fan-out-on-read (pull model):** do nothing at write time; when a follower opens their feed, pull posts from everyone they follow. Cheap writes, slower reads. Instagram uses a hybrid: push for normal users, pull for celebrities.

### TTL (Time To Live)
An expiry timer on a piece of data. A cache entry with TTL = 60 seconds auto-deletes after 60 seconds. A seat lock with TTL = 600 seconds releases the seat after 10 minutes if the user doesn't complete checkout. Every distributed lock should have a TTL — otherwise one crashed client permanently holds the lock.

### Backpressure
When a downstream system can't keep up, it pushes resistance back upstream. Instead of crashing, the system slows down the producer. Kafka does this naturally — if consumers are slow, messages pile up in the topic (bounded by retention). Token buckets for provider rate limiting (Notification System) are another form: workers can't send faster than the provider can absorb.

### Cursor-Based Pagination
"Give me 20 rows after this specific ID" instead of "skip the first 500 rows, give me the next 20." Offset-based pagination is O(N) — the database still scans through all skipped rows. Cursor-based is O(1) — it's a direct index jump. At scale, the difference is night and day. Every feed API in your notes uses cursor pagination.

---

## Messaging & Event Systems

### Kafka (Apache Kafka)
A distributed, persistent message queue (event bus). Services publish events ("order created," "payment completed") to Kafka topics; other services consume them asynchronously. Decouples producers from consumers — if the consumer is slow, events just pile up in Kafka (it handles that beautifully). Handles millions of events/sec.

Key concept: **separate Kafka topics per priority.** If critical payment alerts and marketing emails share a topic, a 10M marketing blast blocks the payment alert in the queue.

### Pub/Sub (Publish/Subscribe)
A messaging pattern where a sender publishes a message to a channel without knowing who's listening. All subscribers to that channel get the message. Redis Pub/Sub is the in-memory, low-latency variant. Google Cloud Pub/Sub is the managed, durable variant.

### Dead Letter Queue (DLQ)
A separate queue where permanently failed messages go after exhausting all retries. Instead of losing the message or blocking the main queue, it's parked in the DLQ for manual investigation. Like an "undeliverable mail" bin at the post office.

### Transactional Outbox Pattern
The fix for the dual-write problem. Instead of writing to the database AND publishing to Kafka separately (where one can fail without the other), you write the event into an `outbox` table inside the same database transaction as the real change. A background CDC worker reads the outbox and publishes to Kafka. Since both writes are in one transaction, they commit together or roll back together — no split-brain.

### CDC (Change Data Capture)
A background worker (like Debezium) that tails a database's internal transaction log and streams every change to another system (Kafka, Elasticsearch, etc.) without touching the main write path. The database doesn't even know it's being watched.

### At-Least-Once vs Exactly-Once vs At-Most-Once Delivery
- **At-most-once:** send and forget. If it fails, don't retry. You might lose messages. (UDP, fire-and-forget telemetry)
- **At-least-once:** retry on failure. You'll never lose a message, but you might deliver duplicates. (Kafka default, notification systems)
- **Exactly-once:** every message is delivered exactly one time. Theoretically impossible across unreliable networks and third-party providers. Approximated by combining at-least-once delivery with client-side deduplication (idempotency keys).

Your Notification System uses at-least-once + client-side dedup. Stripe uses idempotency keys for the same reason.

### Webhook
An HTTP callback — instead of you polling a service asking "any updates?", the service calls YOUR endpoint when something happens. SendGrid calls your webhook when an email bounces. Stripe calls your webhook when a payment succeeds. You register a URL, they POST to it. It's the server-to-server version of push notifications.

### Fire-and-Forget
Sending data without waiting for an acknowledgement. Acceptable when the data is ephemeral and the next update arrives soon anyway. Uber uses fire-and-forget for driver GPS updates — if one is lost, the next one comes in 3 seconds. Notification telemetry events work the same way.

---

## Networking & Protocols

### Load Balancer
Distributes incoming traffic across multiple servers so no single server gets overwhelmed.

- **L4 (Layer 4):** routes using only IP/port info — never reads the request body. Extremely fast, handles DDoS-level traffic. The outer shield.
- **L7 (Layer 7):** actually reads the HTTP path (`/upload` vs `/feed`) and routes smartly — heavy uploads go to beefy servers, light API reads go to lightweight servers. Also handles SSL termination.

### API Gateway
The single front door for all client requests. Handles authentication, rate limiting, request routing, and sometimes response caching — before a request ever reaches a real service. Think of it as a receptionist who checks your ID, makes sure you're not coming back too often, and then directs you to the right department.

### Anycast DNS
The same IP address is announced from data centers worldwide. When a user resolves a domain, they're automatically routed to the nearest data center. Like dialing one pizza chain phone number that always rings the closest branch.

### BGP (Border Gateway Protocol)
The routing protocol that makes Anycast work. BGP is how routers on the internet decide "which path should this packet take?" When multiple data centers announce the same IP via BGP, each internet router picks the closest one. If a data center goes down, BGP automatically reroutes traffic to the next closest — no DNS change needed, no client-side logic. Your YouTube note uses Anycast BGP for RTMP ingestion failover: if Ingestion Server A dies, BGP redirects the stream to Server B within 500ms.

### WebSocket
A persistent, bidirectional TCP connection between client and server. Once established, both sides can push data instantly without reconnecting or sending heavy HTTP headers. Each WebSocket frame has ~2 bytes of overhead vs ~800 bytes for an HTTP request. Mandatory for real-time chat, live cursors, live tracking.

### Long Polling
The poor man's WebSocket. The client sends a request, and the server holds it open (doesn't respond) until it has new data to send, or a timeout expires. Then the client immediately sends another request. Less efficient than WebSockets (new HTTP request each cycle) but works through firewalls/proxies that block WebSockets. Used as a fallback.

### SSE (Server-Sent Events)
A one-way push channel from server to client over a standard HTTP connection. Unlike WebSocket (bidirectional), SSE is server-to-client only. Simpler to set up, works through all proxies, auto-reconnects on disconnect. Good for live dashboards, notification feeds, stock tickers — anything where the client listens but doesn't send.

### HTTP/2 Multiplexing
Old HTTP (1.1) needed a separate connection for each resource. HTTP/2 lets dozens of requests/responses fly over a single connection simultaneously — like a multi-lane highway instead of a single lane. This is why scrolling through image-heavy feeds feels fast.

### gRPC
Google's RPC framework built on HTTP/2 and Protocol Buffers. Services talk to each other using strongly-typed function calls instead of REST/JSON. Much faster than REST (binary serialization, multiplexed streams, bidirectional streaming). Used for internal service-to-service communication where speed matters more than human-readability. Not great for browser clients (limited browser support).

### RTMP (Real-Time Messaging Protocol)
A continuous, low-overhead pipe designed for streaming raw live video from software like OBS to ingestion servers. Establishes one persistent TCP connection and streams frames continuously without waiting for acknowledgements.

### QUIC (HTTP/3 over UDP)
A modern protocol that fixes TCP's head-of-line blocking problem. If one packet is lost, the rest keep flowing — unlike TCP where everyone waits for the retransmission. Also bakes encryption into the initial handshake (0-1 round trips instead of 3).

### Head-of-Line Blocking
A TCP problem. If packet #1 is lost, packets #2, #3, #4 all wait behind it even though they arrived fine — like a single-lane highway where one broken-down car blocks everyone behind it. QUIC solves this by using independent streams — one lost packet doesn't block others. This is why QUIC is preferred for mobile streaming on flaky connections.

### HLS / DASH / LL-HLS
Streaming protocols that chop video into small HTTP-downloadable chunks with a manifest (playlist) file listing available qualities. **LL-HLS** (Low-Latency HLS) uses tiny 1-second chunks for near-live delivery. Because they're plain HTTP, CDNs can cache and serve them — unlike WebSocket streams.

### SSL / TLS Termination
Decrypting HTTPS traffic at the load balancer or API gateway so backend services don't have to. The client's encrypted connection ends ("terminates") at the edge, and internal traffic flows unencrypted over the private VPC. This offloads CPU-heavy cryptographic work from every backend server to one dedicated layer.

### Pre-signed URL
A temporary, one-time-use "permission slip" your backend generates so the client can upload a file directly to S3 without routing through your API servers. This is why a 100MB video upload doesn't clog your main servers — the file goes straight to S3.

### VPC (Virtual Private Cloud)
Your own private, isolated network inside AWS/GCP. Data moving between your services over the VPC never touches the public internet — fast, free (no egress charges), and secure. Like an internal hallway vs walking outside.

### Reverse Proxy
A server that sits in front of your backend servers and forwards client requests to them. The client never talks to the backend directly. Nginx and HAProxy are common reverse proxies. It handles load balancing, SSL termination, caching, and compression. Every API Gateway is essentially a fancy reverse proxy with auth and rate limiting bolted on.

### JWT (JSON Web Token)
A self-contained authentication token. Instead of checking a database on every request ("is this session valid?"), the server encodes the user's identity + expiry into a signed token. The client sends this token with every request. The server verifies the signature without hitting any database — stateless auth. The trade-off: you can't revoke a JWT before its expiry without additional infrastructure (blacklist in Redis).

---

## Rate Limiting

### Token Bucket
A rate-limiting algorithm. Imagine a bucket holding up to N tokens, refilling at a fixed rate (e.g., 10 tokens/sec). Each request costs one token. Bucket empty = request rejected. Allows controlled bursts (if the bucket is full, N requests can fire at once, then you're back to the refill rate). **This is what AWS API Gateway and Stripe use.**

### Leaky Bucket
Like the token bucket, but the output rate is perfectly smooth — no bursts. Requests enter a FIFO queue, the queue drains at a constant rate. If the queue is full, new requests are dropped. Use when you need a rock-steady request rate to protect a fragile downstream system.

### Sliding Window Counter
Counts requests in a rolling time window using weighted averages of adjacent fixed windows. O(1) memory, ~98-99% accurate. **This is what most production systems use for general-purpose rate limiting.**

### Sliding Window Log
Stores the timestamp of every single request in a sorted set. Perfectly accurate, zero boundary burst issues, but memory-hungry (O(N) per user). Use for low-QPS, precision-critical limits like paid API tiers.

### Fixed Window Counter
Simplest algorithm — counts requests in clock-aligned intervals (12:00-12:01, 12:01-12:02). Fast, minimal memory, but has the **boundary burst problem:** user sends 100 requests at 12:00:59 and 100 at 12:01:00 = 200 requests in 2 seconds, both technically "within limits."

### 429 Too Many Requests
The HTTP status code you return when a client exceeds the rate limit. Always pair it with a `Retry-After` header so the client knows when to try again. A rate limiter that says "no" without saying "try again in X seconds" is a bad API.

### Fail-Open vs Fail-Closed
If the rate limiter itself goes down:
- **Fail-open:** allow requests through. Prioritizes availability over protection. Use for non-critical endpoints.
- **Fail-closed:** block everything. Prioritizes protection over availability. Use for payment/auth endpoints.

---

## Distributed Systems Patterns

### Consistent Hashing
A technique for distributing data across nodes where adding or removing a node only redistributes a fraction of the keys (instead of reshuffling everything). Used by load balancers, distributed caches, and Kafka partition assignment.

### Sharding (Partitioning)
Splitting data across multiple database instances. **Horizontal sharding** = different rows on different servers (user IDs 1-1M on shard 1, 1M-2M on shard 2). The partition key determines which shard a piece of data lives on. Pick the wrong partition key and you get hot partitions.

### Key Salting
Appending a random bucket number to a database key to spread writes across multiple nodes. If `stream_id = "abc"` causes a hot partition, use `stream_id#0` through `stream_id#99` to distribute across 100 nodes. On read, query all 100 buckets in parallel and merge.

### Idempotency / Idempotency Key
Making an operation safe to retry. If a user's flaky connection sends the same payment request 3 times, an idempotency key (a UUID attached to the request) ensures only the first one processes — the other two are recognized as duplicates and return the original result. The key rides in the HTTP header, not the body.

### Circuit Breaker
A pattern that stops calling a failing downstream service after too many errors. After N failures within a time window, the breaker "opens" — all subsequent calls fail immediately without even trying. After a cooldown, it lets one "probe" request through. If the probe succeeds, the breaker closes and normal traffic resumes. Prevents cascading failures.

### Exponential Backoff + Jitter
Retrying with increasing wait times (1s → 2s → 4s → 8s...) plus a random offset so retries from different clients don't all hit at the same instant. Without jitter, all failed requests retry at exactly the same intervals, creating mini-thundering herds.

### Bloom Filter
A space-efficient probabilistic data structure that answers "is this item in the set?" It can say:
- **"Definitely not in the set"** — 100% accurate.
- **"Probably in the set"** — has a small false positive rate (~1%).

Uses ~12 KB per user vs 80 KB+ for raw arrays. Used for "already swiped" lists (Tinder), "already visited" URLs (web crawlers), "already recommended" content.

### Snowflake / UUIDv7 IDs
Time-sortable, globally unique IDs generated without a central coordinator. The ID embeds a timestamp + machine ID + sequence counter. Because the timestamp is baked in, you can sort by ID and get chronological order for free — no server clock sync needed.

### CQRS (Command Query Responsibility Segregation)
Routing 100% of reads to one system (Elasticsearch, Redis) and 100% of writes to another (Postgres). Never let search traffic and financial writes compete for the same database engine. BookMyShow uses this — browsing hits Elasticsearch, checkout hits Postgres.

### Paxos / Raft (Consensus Algorithms)
Algorithms that let a cluster of servers agree on a value even if some servers crash. Used for leader election, distributed locks, and replicated state machines. **Raft** is the easier-to-understand version — it elects a leader, and the leader replicates decisions to followers. If the leader dies, a new election happens in seconds. Cassandra uses Paxos for lightweight transactions. etcd and Consul use Raft.

### Quorum
The minimum number of nodes that must agree before a read/write is considered successful. In a cluster of 5 nodes, a quorum is 3 (majority). **Quorum writes** (W=3) + **quorum reads** (R=3) guarantee you always read the latest write (because W + R > N). Cassandra lets you tune W and R per query — strong consistency when W + R > N, eventual consistency when they're lower.

### Leader Election
The process of picking one node in a cluster to be the "leader" (primary) that handles writes, while others are followers (replicas). If the leader crashes, followers detect it via heartbeat timeouts and hold a new election (using Raft/Paxos). Redis Sentinel and Postgres patroni do this automatically.

### SELECT ... FOR UPDATE
A SQL command that acquires an exclusive row-level lock during a transaction. Any other transaction trying to read or modify that row has to wait until the lock is released. Used in BookMyShow (lock the seat row during checkout) and Stripe (lock the balance row during transfer). It's the Postgres-level backstop behind the Redis SETNX fast path.

### RBAC (Role-Based Access Control)
Assigning permissions based on roles, not individual users. Instead of "User X can INSERT into transactions table," you say "the `api_service` role can INSERT; User X has the `api_service` role." In Stripe's design, production service accounts get INSERT/SELECT only — the database engine itself rejects UPDATE/DELETE from those accounts, even if the account is compromised.

### Authorization Hold (Pre-Auth)
A temporary hold on a credit card that verifies the card is real and has funds without actually charging it. The $0 or $1 hold appears as "pending" and auto-releases. BookMyShow uses this at seat-lock time to filter out bots that don't have a real payment method — it's the strongest anti-abuse mechanism because it puts a real cost behind the lock.

### Hash Chaining
Each record stores a SHA-256 hash of its own data PLUS the previous record's hash — like a blockchain. If someone tampers with record #10, every hash after it breaks. A background scanner catches the broken chain instantly. Used in Stripe's payment ledger for tamper-evidence on financial records.

### Saga Pattern
A way to handle distributed transactions across multiple services without 2PC. Instead of one big ACID transaction, you break it into a chain of local transactions, each with a compensating action (rollback). If step 3 fails, you run the compensating actions for steps 2 and 1. Example: "charge card → reserve inventory → confirm order." If inventory reservation fails, refund the card. Eventual consistency, but no distributed locks.

### Two-Phase Commit (2PC)
A distributed transaction protocol: Phase 1 (prepare) — coordinator asks all participants "can you commit?" Phase 2 (commit) — if everyone said yes, coordinator tells everyone to commit. If anyone said no, everyone rolls back. The problem: if the coordinator crashes between phases, everyone is stuck waiting — it's a blocking protocol. That's why most modern systems avoid 2PC and use sagas or outbox patterns instead.

---

## Specific Technologies & Concepts

### Open Connect (Netflix CDN)
Netflix's custom CDN. They ship hardware appliances loaded with SSDs directly into ISP data centers worldwide. The ISP hosts the box for free (because Netflix traffic = ~15% of all downstream internet in North America, and keeping it local saves the ISP's backbone bandwidth). 95%+ of all Netflix video never touches the public internet.

### DRM (Digital Rights Management)
Encrypting video content so only authorized, paid devices can play it. All video chunks on the CDN are AES-encrypted ciphertext — useless without the decryption key. When you press Play, your device proves it's a paid subscriber and gets a short-lived key that lives in a hardware-protected enclave (the app never sees the raw key).

- **Widevine:** Google's DRM (Android, Chrome)
- **FairPlay:** Apple's DRM
- **PlayReady:** Microsoft's DRM

### Adaptive Bitrate Streaming (ABR)
The video player automatically switches between quality levels (4K → 1080p → 720p → 480p) based on current network speed. The server pre-encodes the video into all these quality levels. The player picks the best one it can handle right now — no manual "set quality" needed.

### Per-Title Encoding
Netflix's approach: instead of encoding every movie at the same fixed bitrate, analyze each title's visual complexity scene by scene. A dark dialogue scene gets fewer bits (still looks perfect). An explosion scene gets more. Same perceived quality at 20-50% less bandwidth. At 100 Tbps of global egress, this saves hundreds of millions per year.

### Geohash
Encoding a GPS coordinate (lat, lng) into a single string where nearby points share a common prefix. This converts a 2D proximity problem into a 1D string comparison — enabling fast "find nearby" lookups in databases that only understand string comparisons.

### H3 Hexagonal Grid (Uber)
Uber's open-sourced system that divides the Earth's surface into hexagonal cells at multiple resolutions. "Find nearby drivers" = look up the hex cell containing the pickup point + its 6 neighbors = 7 bucket lookups. **Why hexagons over squares?** Every hex neighbor is equidistant from the center — no corner-distortion problem.

### Google S2 Geometry
Google's alternative to H3. Projects the globe onto a cube face and fills it with a Hilbert curve, converting 2D spatial proximity into 1D sort order. Used by Google Maps.

### PostGIS
A PostgreSQL extension that adds spatial indexing (R-Tree) for geographic queries. Works great up to ~10K QPS but can't handle millions of location writes/sec because it's disk-backed. Used as a fallback for complex polygon queries (geofencing, zone boundaries) that Redis can't express.

### Operational Transformation (OT)
An algorithm for collaborative editing. When two people type at the same spot, the server adjusts the second person's edit to account for what the first person changed. Requires a central server (the "sequencer") to decide canonical operation order. Google Docs uses this — their engine is called **Jupiter**.

### CRDT (Conflict-free Replicated Data Type)
An alternative to OT. Each character gets a unique, globally-ordered ID (not an integer position). Operations never conflict because they reference unique IDs. No central server needed — every client can apply operations in any order and still converge. More memory overhead than OT, but mathematically proven to converge. **Figma uses CRDTs.** Yjs is the most popular open-source CRDT library (used by Notion, Jupyter).

### Fractional Indexing
Assigning positions between existing items (e.g., position 1.5 between 1 and 2) so insertions never shift existing positions. Used in CRDTs to avoid the "my position 10 is now your position 11" problem.

### Commutativity
A mathematical property where operations can be applied in any order and produce the same result. `A + B = B + A` → addition is commutative. This is the key property that makes CRDTs work — if operations commute, it doesn't matter what order different clients receive them in.

### Operation Log (Op Log)
An append-only record of every edit ever made to a document. The document is a *materialized view* of this log — you can reconstruct any version by replaying the log from the start. Google Docs and every collaborative editor uses this. Combined with periodic snapshots, it enables version history without replaying millions of operations.

### Snapshot (Periodic Checkpoint)
A full "photo" of the current state at a point in time. Instead of replaying the entire operation log from document creation, load the nearest snapshot and replay only the operations after it. Google Docs takes snapshots every ~1,000 operations or every 5 minutes. Same concept as database snapshots for backup/recovery.

### Presence
Real-time awareness of who else is in a shared context — live cursors in Google Docs, colored selections, "User X is typing" labels. Presence data is high-frequency (cursor moves every 50ms) and ephemeral (no need to persist) — stored in Redis, pushed via WebSocket Pub/Sub.

### Sequencer
The central server component in OT (Operational Transformation) that assigns a canonical order to concurrent operations. When two edits arrive at the same time, the sequencer decides which is "first" and transforms the other accordingly. It's the single point of serialization — if it dies, a new server takes over from the persisted operation log.

### ETA (Estimated Time of Arrival)
Computed using live traffic data, road network graphs (Dijkstra/A*), and historical speed patterns — not straight-line distance. A 3 km trip might be 5 minutes or 25 minutes depending on traffic. The ETA service is critical for ride-hailing matching (Uber ranks candidate drivers by ETA, not distance) and food delivery (Zomato matches partners who'll arrive when the food is ready).

### Dispatch / Matching
The algorithm that picks the best available driver/partner for a request. Not just "nearest" — considers ETA, driver rating, current direction of travel, and whether the driver is about to complete another trip. The dispatch race condition (two riders matched to the same driver) is solved with atomic claim via Redis SETNX + Postgres backstop.

### Chaos Monkey / Chaos Engineering
Netflix's practice of intentionally killing production servers to test system resilience. Chaos Monkey randomly terminates EC2 instances during business hours. Chaos Kong simulates an entire AWS region failure. If you haven't tested your failover, you don't have failover.

### Surge Pricing
Dynamically raising prices when demand exceeds supply in a geographic zone. Uber tracks demand/supply per H3 hex cell in Redis using atomic counters. When the ratio crosses a threshold, a surge multiplier kicks in. The price increase is a market mechanism to rebalance driver distribution, not just profit extraction.

### Geofencing
Defining virtual geographic boundaries (airport pickup zones, city limits, surge zones) and triggering actions when a device enters or exits them. Used for location-specific rules that can't be expressed with a simple radius query.

### Order State Machine
A sequence of states an order transitions through (REQUESTED → MATCHED → PICKED_UP → IN_TRANSIT → DELIVERED). Each transition triggers different downstream actions. Invalid transitions are rejected (you can't go from DELIVERED back to REQUESTED). Used everywhere from Uber rides to Zomato orders to BookMyShow tickets.

### Signal Protocol (E2EE)
WhatsApp's end-to-end encryption. The server is a blind router — it moves ciphertext without ever holding a decryption key.
- **X3DH handshake:** Alice derives a shared secret using Bob's public keys, without Bob being online.
- **Double Ratchet:** every message advances the key chain, deriving a fresh AES-256 key. Compromising Bob's phone tomorrow doesn't reveal today's messages because today's keys were already destroyed (**Perfect Forward Secrecy**).

### Multi-Armed Bandit
An algorithm that balances exploring new options (trying different thumbnails) with exploiting known winners (showing the best-performing one). Netflix uses this for personalized artwork — serve different thumbnail images to different users for the same movie, converge on what drives the most clicks per user segment.

### A/B Testing
Showing two variants of a feature to different user groups and measuring which performs better. Netflix A/B tests thumbnail images per user segment. The multi-armed bandit is a smarter version that dynamically shifts traffic toward the winning variant instead of waiting for the test to end.

### Notification Digest / Aggregation
Batching multiple low-priority notifications into a single summary: "Rahul, Priya, and 3 others liked your post" instead of 5 separate push notifications. Your phone buzzes once instead of five times. Controlled by a per-user sliding window rate limit in Redis — if the hourly cap is hit, non-critical notifications are held in a buffer and drained as one aggregated message.

### Fallback Escalation
Automatically trying a different channel when the primary channel fails. If push delivery fails after 2 retries for a CRITICAL notification, auto-escalate to SMS — even if the user didn't explicitly opt into SMS for that type. A security alert reaching the user via SMS is better than not reaching them at all.

### Collaborative Filtering
"Users like you also watched…" — recommending based on behavior patterns of similar users. Uses matrix factorization or neural collaborative filtering on a user × item interaction matrix. Needs some history to work (useless for brand new users).

### Content-Based Filtering
Recommending based on the content's own attributes (genre, cast, director, tags) rather than other users' behavior. Works even for new users with zero history — you can always fall back to globally popular + regionally trending content.

### Cold Start Problem
A new user with zero history. The recommendation engine has nothing to personalize with. Fix: start with content-based filtering, transition to collaborative filtering after 2-3 interactions, then blend both (hybrid model).

---

## Infrastructure Components

### APNs (Apple Push Notification service)
Apple's server that delivers push notifications to iPhones/iPads. Requires persistent HTTP/2 connections with TLS client certificates. You maintain a pool of long-lived connections — don't reconnect per push.

### FCM (Firebase Cloud Messaging)
Google's equivalent of APNs. Delivers push notifications to Android devices and web browsers. Also supports "topic" notifications — publish to a topic, all subscribed devices receive it, offloading fan-out to Google.

### Device Token
A unique identifier assigned by APNs/FCM to a specific app installation on a specific device. Used to route push notifications to the right phone. Tokens go stale when users uninstall the app, switch phones, or disable permissions — you need to prune them.

### Debezium
The most common open-source CDC tool. Tails a database's transaction log (Postgres WAL, MySQL binlog) and streams every change to Kafka. Powers the transactional outbox pattern.

### Zuul (Netflix)
Netflix's open-sourced API gateway. Handles dynamic routing, load shedding, and authentication at the edge. At one point, all Netflix API traffic passed through Zuul.

### Ringpop (Uber)
Uber's open-sourced consistent hashing library that assigns geographic zones to specific dispatch servers. If a server dies, the hash ring redistributes its zones to neighbors — no central coordinator needed.

---

## Architecture Patterns (Quick Reference)

| Pattern | One-Line Summary |
|---|---|
| **Polyglot Persistence** | Use the right database for each access pattern — SQL for transactions, NoSQL for time-series, Redis for cache, S3 for blobs |
| **Stateless Compute** | App servers store zero local state. All state lives in shared databases/caches. This is how you auto-scale behind a load balancer |
| **Async Pre-Computation** | Heavy read queries are computed in the background and cached. User gets instant results from cache, not a live DB query |
| **Write Buffering** | Put a message broker (Kafka) in front of databases during write spikes. Absorb the burst, drain at a safe rate |
| **Hybrid Push-Pull** | For celebrity fan-out: push to followers of normal users (small fan-out), pull at read time for celebrity content (avoids millions of writes) |
| **Two-Layer Locking** | Redis `SETNX` as the fast first check, Postgres `SELECT FOR UPDATE` as the ACID backstop. Redis is the bouncer, Postgres is the legal contract |
| **Balance Slotting** | Split a hot account's balance across N rows to spread write contention. Read = `SUM()` across all slots |
| **Admission Control** | Deliberately slowing how fast traffic reaches your backend — a virtual waiting room, not just rate limiting |

---

*Last updated: 2026-07-16*
