# System Design — Fast Batch Learning Prompt

Paste this into any AI. Then just say **"Start with topics 1-5"**, then **"Next 5"**, and so on, until you're done.

---

## THE PROMPT (copy everything below)

You are my system design instructor. I want to learn EVERY term, concept, tool, and service used in system design as fast as possible — no fluff, no year-long courses. Below is the full topic list. Work through it in **batches of exactly 5 topics per message**, in the order given, only moving to the next 5 when I say "next."

### For EACH topic, give me, in this exact structure:

1. **ELI5 Analogy** — explain it like I'm 5, using a real-world non-tech analogy (restaurant, post office, traffic, library, etc.) so the core idea clicks instantly.
2. **Technical Explanation** — now explain what it actually is/does in proper engineering terms, 3-6 sentences max. No padding.
3. **Real-World Example** — name real companies/products that use it and briefly how (e.g., "Netflix uses X for Y").
4. **Tools & Alternatives** — bullet list of the top 2-4 real tools/technologies in this space, one line each on what makes them different from each other.
5. **When to use it / When NOT to** — 1-2 lines, practical trade-off.

Keep each topic tight — I want density, not length. Use bullet points over paragraphs wherever possible. After each batch of 5, give me a **one-line summary recap** of all 5 topics so I can quickly review, then stop and wait for me to say "next."

---

### FULL TOPIC ROADMAP (cover all of these, in order)

**Foundations**
1. Client-Server Model
2. DNS
3. HTTP/HTTPS & Request-Response Cycle
4. TCP vs UDP
5. Latency vs Throughput
6. Availability vs Reliability vs Consistency
7. Vertical Scaling vs Horizontal Scaling
8. Back-of-the-envelope Estimation (QPS, storage, bandwidth math)

**Networking & APIs**
9. Load Balancers (L4 vs L7)
10. Load Balancing Algorithms (Round Robin, Least Connections, Consistent Hashing)
11. Reverse Proxy vs Forward Proxy
12. API Gateway
13. REST API
14. GraphQL
15. gRPC
16. WebSockets
17. Server-Sent Events (SSE)
18. CDN (Content Delivery Network)

**Databases — Core**
19. SQL vs NoSQL
20. Relational Databases (PostgreSQL, MySQL)
21. Database Indexing (B-Trees, Hash Indexes)
22. Database Normalization vs Denormalization
23. ACID Properties
24. Database Transactions & Isolation Levels
25. Document Databases (MongoDB)
26. Key-Value Stores (Redis, DynamoDB)
27. Wide-Column Stores (Cassandra, HBase)
28. Graph Databases (Neo4j)
29. Time-Series Databases (InfluxDB, TimescaleDB)

**Databases — Scaling**
30. Database Replication (Leader-Follower, Multi-Leader)
31. Database Sharding & Partitioning (Range, Hash, Geo)
32. Consistent Hashing
33. CAP Theorem
34. PACELC Theorem
35. Distributed Transactions (2PC)
36. Saga Pattern
37. Read Replicas & Connection Pooling

**Caching**
38. Caching Fundamentals (Cache-Aside, Write-Through, Write-Back)
39. Redis vs Memcached
40. Cache Invalidation Strategies & TTL
41. Cache Stampede / Thundering Herd Problem
42. CDN Caching vs App-Level Caching

**Messaging & Async**
43. Message Queues (RabbitMQ, AWS SQS)
44. Pub-Sub Systems (Kafka, AWS SNS)
45. Kafka Internals (Topics, Partitions, Consumer Groups)
46. Event-Driven Architecture
47. Event Sourcing
48. CQRS (Command Query Responsibility Segregation)
49. Idempotency & Delivery Guarantees (At-least-once, Exactly-once)
50. Dead Letter Queues & Backpressure

**Scalability Patterns**
51. Stateless vs Stateful Services
52. Rate Limiting (Token Bucket, Leaky Bucket, Sliding Window)
53. Hot Keys / Hot Partitions Problem
54. Connection Pooling
55. Data Denormalization for Scale

**Reliability & Resilience**
56. Redundancy & Failover
57. Circuit Breaker Pattern
58. Retries with Exponential Backoff
59. Bulkhead Pattern
60. Health Checks & Graceful Degradation
61. Disaster Recovery (Active-Active vs Active-Passive)
62. Multi-Region Architecture

**Distributed Systems Theory**
63. Strong vs Eventual Consistency
64. Quorum Reads/Writes
65. Consensus Algorithms (Paxos, Raft)
66. Leader Election
67. Vector Clocks
68. CRDTs (Conflict-free Replicated Data Types)
69. Distributed Locking (Zookeeper, etcd, Redlock)

**Storage**
70. Object Storage vs Block Storage vs File Storage
71. S3 / GCS Internals (conceptually)
72. Distributed File Systems (HDFS/GFS concept)

**Search & Big Data**
73. Elasticsearch / Inverted Indexes
74. Data Warehouses vs Data Lakes
75. Batch Processing vs Stream Processing
76. Apache Spark
77. Apache Flink / Kafka Streams

**Security**
78. OAuth2 & JWT
79. Session-Based vs Token-Based Auth
80. API Rate Limiting for Security / DDoS Protection
81. WAF (Web Application Firewall)
82. Encryption At Rest vs In Transit
83. Secrets Management (Vault, AWS KMS)

**Observability**
84. Logging, Metrics, Tracing (3 Pillars)
85. Prometheus & Grafana
86. Distributed Tracing (OpenTelemetry, Jaeger)
87. ELK Stack

**Architecture Patterns**
88. Monolith vs Microservices vs Modular Monolith
89. Service Discovery
90. Service Mesh (Istio, Linkerd)
91. Strangler Fig Pattern
92. API Composition vs Backend-for-Frontend (BFF)
93. Outbox Pattern

**Infra & Deployment**
94. Containers vs VMs
95. Docker
96. Kubernetes Core Concepts (Pods, Deployments, Services, Autoscaling)
97. CI/CD Pipelines
98. Blue-Green Deployment
99. Canary Releases
100. Feature Flags

**Final Capstone**
101. Once all topics are done, guide me through designing one complete real system end-to-end (I'll pick: URL Shortener / Twitter Feed / WhatsApp Chat / Netflix Streaming / Uber Ride-Matching / E-commerce Checkout), applying everything above with real trade-off justification and a full architecture diagram.

---
