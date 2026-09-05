# Batch 8: Topics 36–40 (Databases Scaling & Caching)

---

### 36. Saga Pattern

**1. ELI5 Analogy**
* Booking a vacation trip (Flight + Hotel + Rental Car): You book the flight first; then you book the hotel; then you try to book the car. If the car rental company is completely sold out, you don't stay stuck—you automatically call the hotel to cancel your room reservation, and call the airline to refund your flight ticket.

**2. Technical Explanation**
* The Saga pattern manages distributed transactions across multiple microservices without locking resources using a sequence of local database transactions. Each step in the saga executes and commits a local transaction inside its own service boundary and publishes an event or message triggering the next step. If any intermediate local transaction fails (e.g., payment declined or inventory out of stock), the saga executes a series of **compensating transactions** in reverse order to semantically undo changes and restore data consistency. Sagas are coordinated either via **Choreography** (services publish and listen to domain events asynchronously without a central orchestrator) or **Orchestration** (a dedicated saga orchestrator state machine directs services on what transactions to execute).

**3. Real-World Example**
* **Uber / Amazon Order Checkout**: Order Service creates pending order $\to$ Payment Service charges credit card $\to$ Driver/Delivery Service assigns dispatch. If dispatch fails, a compensating refund transaction triggers on Payment Service, and the order marks as canceled.

**4. Tools & Alternatives**
* **Temporal.io / Cadence**: Code-as-configuration workflow orchestrators that guarantee durable state execution and automatic compensating saga rollbacks.
* **Camunda / Zeebe**: BPMN-driven workflow engines built specifically for coordinating complex enterprise saga orchestrations.
* **Event-Driven Choreography (Kafka / RabbitMQ)**: Decentralized messaging where services react to failure events directly without central orchestration.
* **Two-Phase Commit (2PC)**: Tightly coupled, synchronous alternative offering immediate ACID consistency but severe locking overhead.

**5. When to use it / When NOT to**
* **Use when**: Long-running business processes span multiple distinct microservices or databases where holding distributed database locks is unacceptable.
* **Do NOT use when**: Workloads are confined to a single database (use standard ACID transactions) or systems require strict immediate isolation (sagas expose dirty reads of intermediate states before compensation completes).

---

### 37. Read Replicas & Connection Pooling

**1. ELI5 Analogy**
* **Read Replicas**: A celebrity author (Primary) writing a new book; millions of fans read printed paperback copies (Replicas) distributed worldwide so the author doesn't have to read aloud to every person individually.
* **Connection Pooling**: A fleet of 10 shared company taxi cabs waiting outside: When an employee needs a ride, they grab an idle cab, take their trip, and immediately return it to the curb for the next colleague, rather than buying a new car for every single trip.

**2. Technical Explanation**
* **Read Replicas** are read-only database copies that continuously sync with the primary writer node via asynchronous or semi-synchronous Write-Ahead Log (WAL) replication, allowing applications to horizontally scale read throughput by offloading queries from the write master. **Connection Pooling** maintains a persistent pool of pre-established, reusable physical TCP connections between backend application servers and the database engine. Creating a new database connection involves expensive CPU/memory overhead (TCP handshake, TLS negotiation, authentication, backend process forking); connection pools eliminate this per-query setup cost and safeguard the database from crashing under connection exhaustion. However, routing reads to replicas introduces replication lag, meaning reads might momentarily return stale data (requiring read-your-own-writes routing back to the primary).

**3. Real-World Example**
* **Shopify / Reddit**: Route heavy write traffic (posts, checkouts) to primary database nodes, while routing massive feed-scrolling read traffic across dozens of read replicas via **PgBouncer** or **ProxySQL** connection pools.

**4. Tools & Alternatives**
* **PgBouncer**: Ultra-lightweight, high-performance connection pooler for PostgreSQL operating in transaction or session pooling mode.
* **ProxySQL**: High-performance, protocol-aware proxy and connection pooler for MySQL supporting automatic read/write query splitting.
* **HikariCP**: Blazing-fast, lightweight JDBC connection pooling library used standard in Spring Boot/Java ecosystems.
* **AWS RDS Proxy**: Fully managed, highly available database proxy that pools connections and preserves connections during failovers.

**5. When to use it / When NOT to**
* **Use when**: Applications experience high read-to-write ratios (>5:1) and high concurrency spikes where opening new database connections threatens database stability.
* **Do NOT use when**: Workloads are strictly write-dominated (replicas add replication load to the primary without offloading writes) or queries require strict zero-lag read consistency immediately after writing (unless routed to primary).

---

### 38. Zero-Downtime Database Migration & Schema Changes on Live Systems

**1. ELI5 Analogy**
* Changing the engine of an airplane mid-flight: You install the new second engine alongside the old one, test that it runs smoothly, transfer fuel lines one by one over to the new engine, and only unbolt and discard the old engine once the new engine is carrying 100% of the flight load.

**2. Technical Explanation**
* Zero-downtime database migration is the disciplined engineering process of altering database schemas (adding columns, renaming fields, altering types, creating indexes) or migrating entire database engines without taking write or read outages. It relies on the **Expand and Contract (Parallel Run)** pattern executed in four distinct phases:
  1. **Expand**: Add the new nullable column/table and create new indexes concurrently (`CREATE INDEX CONCURRENTLY` in Postgres) without locking tables.
  2. **Dual-Write**: Update application code to read from the old schema while writing to both old and new schemas simultaneously.
  3. **Backfill**: Run asynchronous background batch jobs to copy historical records from the old format to the new format.
  4. **Contract**: Switch application reads to the new schema, stop dual-writing, and drop the deprecated old column/table.

**3. Real-World Example**
* **GitHub / Stripe**: Utilize schema change tools like **gh-ost** (GitHub Online Schema Transformations) to perform live MySQL schema migrations on massive tables by tailing the binlog to apply live writes to shadow tables without table locking.

**4. Tools & Alternatives**
* **gh-ost (GitHub)**: Triggerless online schema change tool for MySQL that operates via asynchronous replication binlog streaming.
* **pt-online-schema-change (Percona)**: Trigger-based tool that creates shadow tables and mirrors writes for MySQL.
* **Liquibase / Flyway**: Database migration version control tools managing declarative and imperative SQL migration scripts.
* **Direct DDL (`ALTER TABLE`)**: Dangerous default approach that locks large tables, causing production query timeouts and cascading system outages.

**5. When to use it / When NOT to**
* **Use when**: Operating customer-facing production systems running 24/7 where table locks exceeding a few hundred milliseconds cause production outages.
* **Do NOT use when**: Developing staging/dev environments or pre-launch greenfield projects where scheduling a brief maintenance window is vastly cheaper and simpler than managing multi-phase dual-writes.

---

### 39. Caching Fundamentals (Cache-Aside, Write-Through, Write-Back)

**1. ELI5 Analogy**
* **Cache-Aside**: You check your desk sticky note for a phone number; if it’s not there, you walk to the master phonebook, look it up, and scribble it on your sticky note for next time.
* **Write-Through**: You write a new phone number onto your sticky note AND immediately write it into the master phonebook before doing anything else.
* **Write-Back (Write-Behind)**: You jot notes down rapidly on your sticky note throughout the day and only update the heavy master filing cabinet once every evening in bulk.

**2. Technical Explanation**
* Caching fundamentals govern the data synchronization patterns between high-speed in-memory tiers (e.g., Redis) and persistent storage (e.g., PostgreSQL). In **Cache-Aside (Lazy Loading)**, the application first checks the cache; on a cache miss, it reads from the database, writes the result to the cache, and returns it (resilient against cache failure, but initial reads are slow and risk stale data). In **Write-Through**, the application writes data to the cache, which synchronously writes through to the database before confirming success (high data consistency and cache freshness, but higher write latency). In **Write-Back (Write-Behind)**, the application writes directly to the cache, which acknowledges immediately and flushes updates to the database asynchronously in batches (ultra-fast write performance, but risks permanent data loss if the cache node crashes before flushing).

**3. Real-World Example**
* **Cache-Aside**: **Wikipedia / Reddit**: User profile and article reads hit Redis/Memcached; if missing, data loads from MySQL and caches with a TTL.
* **Write-Back**: **Gaming Leaderboards / IoT Telemetry**: Millions of score updates or sensor pings increment in Redis memory instantly; a background worker batches writes to disk every 10 seconds.

**4. Tools & Alternatives**
* **Redis / Memcached**: The dominant distributed in-memory caching stores.
* **Application In-Memory Caches (Caffeine for Java, Moka for Rust, Guava)**: Sub-microsecond local process memory caches eliminating network hops to Redis.
* **Read-Through / Write-Through Engines (e.g., Hazelcast, Infinispan)**: Distributed data grids where cache clusters encapsulate the database connection directly.

**5. When to use it / When NOT to**
* **Use Cache-Aside when**: System is read-heavy and can tolerate lazy loading and TTL expiration.
* **Use Write-Through when**: Strong consistency between cache and database is required without stale reads.
* **Use Write-Back when**: Write throughput is overwhelmingly high and occasional minor data loss under node failure is acceptable.

---

### 40. Redis vs Memcached

**1. ELI5 Analogy**
* **Memcached**: A minimalist walk-in locker: You put raw labeled boxes in and pull labeled boxes out. It has multiple doors (multi-threaded) so dozens of people can grab boxes at the exact same instant, but it has no shelves, sorting bins, or permanent locks.
* **Redis**: A Swiss Army knife workshop: It only has one master craftsman working inside at a time (single-threaded core), but it has built-in tool racks, sorted file drawers, calculators, and a safe that writes everything into a physical ledger book.

**2. Technical Explanation**
* Both Redis and Memcached are high-performance in-memory key-value data stores, but they diverge in architecture, data structures, and persistence capabilities. **Memcached** is a multi-threaded, pure in-memory cache utilizing a slab allocator to avoid memory fragmentation, optimized exclusively for simple, high-throughput string/blob key-value caching with horizontal client-side sharding. **Redis** features a single-threaded event-driven core architecture (with I/O threading in v6+) supporting rich native data structures (Strings, Lists, Sets, Sorted Sets, Hashes, Bitmaps, HyperLogLogs, Geospatial, Streams). Furthermore, Redis supports disk persistence (RDB snapshots and AOF logs), master-replica replication, automatic failover (Redis Sentinel), distributed clustering (Redis Cluster), and Pub/Sub messaging.

**3. Real-World Example**
* **Memcached**: **Meta (Facebook)**: Runs massive multi-terabyte Memcached clusters purely to cache pre-rendered HTML fragments and serialized database query results at peak multi-million RPS.
* **Redis**: **Twitter / Discord**: Uses Redis sorted sets (`ZSET`) to compute real-time leaderboards, manage user presence (online/offline status), and store message queues.

**4. Tools & Alternatives**
* **Redis**: Feature-rich, in-memory data structure server with persistence, clustering, and pub/sub.
* **Memcached**: High-throughput, multi-core, simple string/object in-memory cache.
* **KeyDB**: Multithreaded open-source fork of Redis delivering 3–5x higher throughput on multi-core servers.
* **Dragonfly**: Modern C++ in-memory store compatible with both Redis and Memcached APIs, utilizing hardware-efficient thread-per-core architecture.

**5. When to use it / When NOT to**
* **Use Memcached when**: You need simple, high-throughput string/blob caching that scales effortlessly across multi-core machines with minimal operational overhead.
* **Use Redis when**: You require complex data structures (sorted sets, hashes), disk persistence, atomic operations, pub/sub messaging, or leaderboards.

---

### One-Line Summary Recap (Topics 36–40)
> **Sagas** coordinate distributed transactions via local steps and compensating rollbacks, **Read Replicas & Connection Pools** offload read load and eliminate TCP socket churn, **Zero-Downtime Migrations** expand and contract schemas safely on live tables, **Caching Patterns** balance data freshness against write speed, and **Redis vs Memcached** contrasts a feature-rich, versatile data structure engine against a lightweight multi-threaded raw cache.
