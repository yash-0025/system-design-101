# Batch 7: Topics 31–35 (Databases Scaling & Distributed Systems)

---

### 31. Database Sharding & Partitioning (Range, Hash, Geo)

**1. ELI5 Analogy**
* Splitting a massive phone directory into smaller books:
  * **Range**: Book 1 for last names A–H, Book 2 for I–P, Book 3 for Q–Z.
  * **Hash**: Rolling a 3-sided die based on a person's Social Security Number and tossing their page into box 1, 2, or 3 so all boxes stay equally thick.
  * **Geo**: Giving European residents an EU phonebook and American residents a US phonebook so people only search locally.

**2. Technical Explanation**
* Sharding is a database architecture pattern that partitions a single logical dataset horizontally across multiple independent physical database instances (shards), each holding a unique subset of the data. In **Range Sharding**, records are partitioned based on contiguous value intervals (e.g., dates or IDs), which allows fast range queries but frequently creates severe write hotspots on the latest range. In **Hash Sharding**, a hash function is applied to a shard key (e.g., `hash(user_id) % num_shards`), distributing writes uniformly across nodes at the expense of scattering range queries across all shards (scatter-gather). In **Geo Sharding**, partitions map directly to user geographic regions, reducing network latency and satisfying data sovereignty mandates (e.g., GDPR).

**3. Real-World Example**
* **Instagram / Discord**: Discord sharded its massive message database (first on MongoDB, then Cassandra, now ScyllaDB) using `guild_id` (server ID) as the shard/partition key, ensuring all messages in a specific Discord server reside on the same node.

**4. Tools & Alternatives**
* **Citus (PostgreSQL extension)**: Transforms Postgres into a horizontally sharded distributed database using hash-distributed tables.
* **Vitess**: Database clustering system that transparently shards MySQL at hyperscale (originally built for YouTube).
* **CockroachDB / TiDB**: Modern distributed SQL databases that implement automatic range-based sharding and rebalancing under the hood.
* **MongoDB Auto-Sharding**: Native document database sharding based on configurable range or hashed shard keys.

**5. When to use it / When NOT to**
* **Use when**: Dataset size or write throughput exceeds the physical CPU, memory, or disk I/O limits of the largest single bare-metal server.
* **Do NOT use when**: Total database size is modest (< a few terabytes) and can be handled by vertical scaling and read replicas, because sharding complicates cross-shard joins, distributed transactions, and schema migrations.

---

### 32. Consistent Hashing

**1. ELI5 Analogy**
* A round circular dinner table with 6 lazy-susan bowls placed evenly around the rim: Whenever someone passes a dish, they slide it clockwise until it reaches the nearest bowl. If you add or remove one bowl, you only move dishes between that bowl and its neighbor, rather than re-sorting every single dish on the entire table.

**2. Technical Explanation**
* Consistent Hashing is a distributed algorithmic hashing technique where both storage nodes (servers) and data keys are mapped onto a fixed circular identifier ring using a uniform hash function (typically $0 \text{ to } 2^{32}-1$). A key is assigned to the first server node encountered moving clockwise along the ring. Unlike traditional modulo hashing (`key % N`), where adding or removing a node changes the modulus and remaps nearly 100% of all keys ($O(K)$), consistent hashing ensures that only $\frac{K}{N}$ keys must be relocated on cluster topology changes. To prevent data skew and hotspots caused by non-uniform server distribution, consistent hashing utilizes **virtual nodes** (vnodes), assigning dozens or hundreds of virtual ring tokens to each physical machine.

**3. Real-World Example**
* **Amazon Dynamo / Apache Cassandra / Discord**: Uses consistent hashing rings with virtual nodes to distribute partitioned partition keys evenly across peer nodes and maintain cache locality during cluster scale-out.
* **Akamai / HAProxy / Envoy**: Routes web requests to cache servers using consistent hashing on client IP or URL to maximize cache hits.

**4. Tools & Alternatives**
* **Ketama**: Standardized, battle-tested consistent hashing library originally developed for Memcached client clustering.
* **Maglev (Google)**: Consistent hashing algorithm designed for Google's software load balancers, featuring $O(1)$ lookup time and minimal reshuffling under failure.
* **Rendezvous Hashing (Highest Random Weight / HRW)**: Alternative to ring-based consistent hashing that computes a hash weight for every (key, node) pair and picks the maximum.

**5. When to use it / When NOT to**
* **Use when**: Building distributed caches, distributed hash tables (DHTs), or partitioned database storage where nodes are added or removed dynamically.
* **Do NOT use when**: The number of storage nodes is completely static and fixed, where simple modulo hashing (`hash(key) % N`) has lower implementation complexity and zero vnode overhead.

---

### 33. CAP Theorem

**1. ELI5 Analogy**
* Ordering food at a drive-thru during a sudden phone line outage between the speaker and the kitchen:
  * You can choose **Availability**: The drive-thru employee sells you whatever food is sitting under the heat lamp, but it might not be what you actually ordered (the kitchen has no clue).
  * Or you can choose **Consistency**: The employee refuses to take your order and shows a "Closed due to technical error" sign until the phone line is fixed, ensuring no mistakes are made.
  * You cannot choose both while the phone line is cut (**Partition**).

**2. Technical Explanation**
* Formulated by Eric Brewer, the CAP Theorem states that in any asynchronous networked distributed data store, it is mathematically impossible to simultaneously provide more than two out of three guarantees: **Consistency (C)** (every read receives the most recent write or an error), **Availability (A)** (every non-error response returned by non-failing nodes, without guaranteeing it contains the latest write), and **Partition Tolerance (P)** (the system continues to operate despite arbitrary packet drops or network partitions between nodes). Because network partitions and hardware communication blips are unavoidable in real-world distributed networks, **Partition Tolerance is mandatory**. Therefore, distributed systems are forced to choose between **CP** (sacrifice availability during partitions to guarantee fresh, identical data) and **AP** (sacrifice consistency to ensure the system always responds, accepting stale or divergent reads).

**3. Real-World Example**
* **CP System**: **Google Cloud Spanner / CockroachDB / ZooKeeper / etcd**: If a network partition isolates a minority quorum, nodes in that partition reject writes and reads to prevent split-brain and stale reads.
* **AP System**: **Apache Cassandra / Amazon DynamoDB / Couchbase**: During network partitions, nodes on both sides continue accepting writes and reads locally, reconciling divergent states asynchronously later using vector clocks or Last-Write-Wins (LWW).

**4. Tools & Alternatives**
* **CP Databases**: etcd, Consul, ZooKeeper, CockroachDB, HBase (enforce consensus via Raft/Paxos).
* **AP Databases**: Cassandra, ScyllaDB, DynamoDB (with eventual consistency), Riak.
* **CA Systems**: Traditional single-instance RDBMS (PostgreSQL, MySQL on a single box) are technically "CA", but only because they ignore distributed networks ($P$) entirely.

**5. When to use it / When NOT to**
* **Choose CP when**: Business requirements strictly forbid stale data or conflicting concurrent writes (financial transactions, inventory reservations, distributed locks).
* **Choose AP when**: Business requires continuous 24/7 uptime and degraded or slightly stale data is completely acceptable (social media comments, DNS resolution, streaming telemetry).

---

### 34. PACELC Theorem

**1. ELI5 Analogy**
* A business meeting between a boss and a remote partner:
  * When the phone line is cut (**P**), do you pause the meeting (**C**) or keep talking anyway (**A**)?
  * But what about regular days when the phone line works perfectly (**Else**)? Do you talk slowly and wait for the partner to confirm every word (**C**), or do you talk at normal speed so the meeting finishes quickly (**Latency**)?

**2. Technical Explanation**
* Proposed by Daniel Abadi in 2012, the PACELC theorem extends the CAP theorem by addressing how distributed databases behave during normal, healthy operations when no network partition exists. The acronym states: If there is a **P**artition, how does your system trade off **A**vailability vs **C**onsistency? **E**lse (under normal operation), how does your system trade off **L**atency vs **C**onsistency? Because network partitions are rare compared to steady-state operations, systems must explicitly decide whether to wait for multi-node round-trip replication to guarantee strict consistency ($C$) or return immediately to minimize client latency ($L$).

**3. Real-World Example**
* **PA/EL (Cassandra, DynamoDB)**: In a partition, choose Availability ($PA$); else, return writes immediately from local memory to minimize Latency ($EL$), replicating in the background.
* **PC/EC (Google Spanner, CockroachDB, ZooKeeper)**: In a partition, choose Consistency ($PC$); else, wait for Raft/Paxos consensus round-trips to maintain strict Consistency ($EC$), accepting higher latency.
* **PA/EC (MongoDB default)**: In a partition, remains Available ($PA$); else under normal operation, waits for majority acknowledgment to guarantee Consistency ($EC$).

**4. Tools & Alternatives**
* **PA/EL Systems**: Apache Cassandra, ScyllaDB, DynamoDB (eventually consistent mode), Couchbase.
* **PC/EC Systems**: CockroachDB, Google Spanner, etcd, Consul.
* **Tunable Systems**: Cassandra and DynamoDB allow client-level overrides per query (`ConsistencyLevel.ONE` for PA/EL vs. `ConsistencyLevel.QUORUM` for PC/EC).

**5. When to use it / When NOT to**
* **Use PACELC when**: Formulating real-world distributed database architectures, because evaluating only CAP ignores the everyday trade-off between client-perceived latency and replication consistency.
* **Do NOT rely purely on CAP when**: Diagnosing steady-state performance bottlenecks, because high database latency is usually an intentional trade-off for EC (Consistency over Latency) rather than a partition event.

---

### 35. Distributed Transactions (2PC - Two-Phase Commit)

**1. ELI5 Analogy**
* A marriage ceremony led by an officiant:
  * **Phase 1 (Prepare / Voting)**: The officiant asks Person A: "Do you take...?", then asks Person B: "Do you take...?". Both must explicitly answer "I do".
  * **Phase 2 (Commit / Execution)**: If both said "I do", the officiant declares "I pronounce you married!" (Commit). If either said "No" (or fainted), the wedding is called off immediately (Abort/Rollback).

**2. Technical Explanation**
* Two-Phase Commit (2PC) is a distributed atomic commitment protocol that guarantees all participating database nodes (cohorts) either commit or abort a transaction together across network boundaries. It is driven by a centralized transaction coordinator in two phases:
  1. **Prepare Phase**: The coordinator issues a `PREPARE` command to all cohorts; each cohort executes the transaction locally up to the commit point, writes undo/redo logs, locks resources, and votes `YES` (ready) or `NO` (abort).
  2. **Commit Phase**: If *all* cohorts vote `YES`, the coordinator persists a commit record to its log and sends `COMMIT` to all nodes; if any cohort votes `NO` or times out, the coordinator broadcasts `ROLLBACK`.
  While 2PC guarantees strict ACID Atomicity across distributed systems, it is a synchronous, blocking protocol: if the coordinator crashes during Phase 2, cohort nodes remain stuck holding exclusive locks indefinitely, severely degrading throughput.

**3. Real-World Example**
* **Inter-Bank Wire Settlements (Fedwire / SWIFT)**: Uses 2PC or Three-Phase Commit (3PC) across multi-institution core banking systems to ensure money is atomically debited from Bank A and credited to Bank B without money vanishing or duplicating.

**4. Tools & Alternatives**
* **XA Transactions (JTA / MySQL XA / PostgreSQL 2PC)**: The open industry standard specification for implementing two-phase commit across heterogeneous relational databases.
* **Three-Phase Commit (3PC)**: Non-blocking extension of 2PC introducing a `PreCommit` phase and timeout aborts, but vulnerable to network partitions.
* **Saga Pattern**: Asynchronous, non-blocking distributed architecture using compensating transactions; widely preferred over 2PC in modern microservices.

**5. When to use it / When NOT to**
* **Use when**: Atomic, zero-error consistency across multiple distinct distributed databases is a strict legal or financial requirement.
* **Do NOT use when**: Building high-scale, low-latency microservice architectures, where 2PC locking creates severe distributed deadlocks, latency spikes, and single-coordinator failure bottlenecks (use the Saga pattern instead).

---

### One-Line Summary Recap (Topics 31–35)
> **Sharding** splits massive tables across physical hardware nodes, **Consistent Hashing** prevents catastrophic key reshuffling when nodes change, **CAP Theorem** forces a choice between availability and consistency during network cuts, **PACELC** extends CAP to everyday latency-vs-consistency trade-offs, and **2PC** coordinates all-or-nothing distributed commits at the cost of synchronous blocking.
