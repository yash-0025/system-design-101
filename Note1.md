# Batch 1: Topics 1–5 (Foundations)

---

### 1. Client-Server Model

**1. ELI5 Analogy**
* Ordering food at a restaurant: You (the client) sit at a table and make an order; the kitchen (the server) prepares the food and sends it back to your table. You don't cook it yourself, and the kitchen only cooks when an order arrives.

**2. Technical Explanation**
* The client-server model is a distributed computing architecture where workload is partitioned between service requesters (clients) and service providers (servers). Clients initiate communication sessions by sending requests over a network protocol to a centralized or distributed host listening on a dedicated port. The server processes incoming requests, enforces business logic, queries persistent data stores, and returns an appropriate response payload. This decouples user interface state and presentation concerns from centralized data processing and security management.

**3. Real-World Example**
* **Uber**: The mobile app (client) requests ride status and driver locations from Uber’s centralized backend microservices (servers), which calculate routing, pricing, and matching before responding.

**4. Tools & Alternatives**
* **REST/HTTP Servers (e.g., Express, FastAPI, Spring Boot)**: Standard request-response servers answering stateless client queries.
* **Peer-to-Peer (P2P) Architecture (e.g., BitTorrent, IPFS)**: Nodes act simultaneously as both client and server, removing centralized single points of failure.
* **Master-Worker / Actor Model (e.g., Akka, Ray)**: Distributed nodes coordinate via leader-driven scheduling rather than consumer-initiated requests.

**5. When to use it / When NOT to**
* **Use when**: You need centralized access control, business logic isolation, consistent data state, and lightweight client runtimes.
* **Do NOT use when**: You need decentralized, fault-tolerant networks with zero central hosting costs or when clients must communicate without an intermediary server (e.g., BitTorrent).

---

### 2. DNS (Domain Name System)

**1. ELI5 Analogy**
* The contacts list in your phone: You don't memorize everyone's 10-digit phone number; you just tap "Mom", and your phone translates that name into the real number to dial.

**2. Technical Explanation**
* DNS is a hierarchical, distributed naming system that translates human-readable domain names (e.g., `google.com`) into machine-routable IP addresses (IPv4/IPv6). Resolution begins at root name servers (`.`), moves down to Top-Level Domain (TLD) servers (`.com`), and queries authoritative name servers for record lookups (A, AAAA, CNAME, MX). DNS heavily leverages multi-tiered caching (browser, OS, local recursive resolvers, ISPs) bounded by Time-To-Live (TTL) values. It also powers global traffic steering via Geo-DNS and latency-based routing.

**3. Real-World Example**
* **Spotify / Cloudflare**: Route global users to the geographically closest data center edge using Latency-Based Geo-DNS routing to reduce round-trip times.

**4. Tools & Alternatives**
* **Amazon Route 53**: Highly scalable managed cloud DNS with health-checking, failover, and weighted latency routing.
* **Cloudflare DNS (1.1.1.1)**: High-performance, privacy-first public recursive resolver and authoritative DNS with instant TTL propagation.
* **CoreDNS**: Extensible, plugin-driven internal DNS server widely used for service discovery inside Kubernetes clusters.
* **Direct IP / Service Registries (Consul, Eureka)**: Internal microservice-to-microservice discovery alternatives bypassing traditional public DNS hierarchies.

**5. When to use it / When NOT to**
* **Use when**: Any public-facing web service requires human-friendly naming, multi-region load steering, or zero-downtime traffic cutovers.
* **Do NOT use when**: You need sub-millisecond dynamic routing for high-churn internal microservices where DNS TTL caching introduces stale routes (use Consul or gRPC service discovery instead).

---

### 3. HTTP/HTTPS & Request-Response Cycle

**1. ELI5 Analogy**
* Sending a letter via postal mail (HTTP) vs. sending it inside an armored, sealed, tamper-proof courier box (HTTPS): The recipient opens it, reads your request, and writes back a reply inside another sealed box.

**2. Technical Explanation**
* HTTP (Hypertext Transfer Protocol) is a stateless application-layer protocol operating over TCP/TLS, where clients send verb-based requests (`GET`, `POST`, `PUT`, `DELETE`) with headers and bodies, receiving numeric status codes (`2xx`, `4xx`, `5xx`) and response payloads. The cycle starts with a TCP handshake, an optional TLS 1.3 cryptographic handshake for HTTPS encryption/identity validation, request parsing, server-side processing, response dispatch, and connection persistence (keep-alive). Modern iterations (HTTP/2, HTTP/3) solve head-of-line blocking via multiplexed binary framing over a single connection and UDP-based QUIC transport.

**3. Real-World Example**
* **Stripe**: Handles millions of transactional payment calls over HTTPS, utilizing TLS encryption and standard HTTP status codes (e.g., `402 Payment Required`, `429 Too Many Requests`) for strict API predictability.

**4. Tools & Alternatives**
* **HTTP/1.1**: Simple text protocol with persistent connections, but suffers from head-of-line blocking on individual TCP streams.
* **HTTP/2**: Binary multiplexing over a single TCP connection with header compression (HPACK) and server push capabilities.
* **HTTP/3 (QUIC)**: Replaces TCP with UDP-based QUIC to eliminate transport-level head-of-line blocking and achieve 0-RTT connection resumption.
* **gRPC / WebSockets**: Alternatives for binary RPC execution or bidirectional long-lived persistent streaming.

**5. When to use it / When NOT to**
* **Use when**: Standard CRUD web APIs, public third-party integrations, and web browser consumption requiring universal interoperability.
* **Do NOT use when**: Ultra-low-latency real-time bidirectional messaging is mandatory (use WebSockets) or high-volume internal microservice RPCs demand raw binary throughput (use gRPC).

---

### 4. TCP vs UDP

**1. ELI5 Analogy**
* **TCP** is a certified courier delivery: You must sign for the package, and if a box is dropped on the highway, the driver stops, picks it back up, and delivers every box in exact numerical order.
* **UDP** is a live megaphone broadcast: The speaker keeps talking without pausing; if you miss three words because a car honked, they won't stop and repeat them.

**2. Technical Explanation**
* TCP (Transmission Control Protocol) is a connection-oriented transport protocol providing reliable, ordered, and error-checked byte stream delivery via a 3-way handshake (`SYN`, `SYN-ACK`, `ACK`), sequence numbers, acknowledgments, and window-based flow/congestion control. UDP (User Datagram Protocol) is a minimal, connectionless, lightweight transport protocol that sends independent packets (datagrams) with zero handshake, ordering guarantees, retransmissions, or flow control. TCP prioritizes data integrity and correctness at the expense of latency and jitter, while UDP minimizes transmission overhead and latency at the cost of potential packet loss.

**3. Real-World Example**
* **TCP**: **GitHub / Banking Apps** use TCP (via SSH/HTTPS) because missing or corrupted git commits or transaction records are intolerable.
* **UDP**: **Discord / Zoom** use UDP for live voice/video streaming because dropping a few audio frames is imperceptible, whereas waiting for retransmissions would cause jarring audio lag.

**4. Tools & Alternatives**
* **TCP**: Standard for Web (HTTP/1.1, HTTP/2), File Transfer (SFTP), Database connections (Postgres, MySQL).
* **UDP**: Standard for DNS lookups, VoIP, live streaming, multiplayer games, and DHCP.
* **QUIC (Quick UDP Internet Connections)**: Google-developed transport layer built on UDP providing TCP-level reliability with multiplexing and zero head-of-line blocking.
* **SCTP (Stream Control Transmission Protocol)**: Message-oriented transport supporting multi-streaming and multi-homing used in telecom systems.

**5. When to use it / When NOT to**
* **Use TCP when**: Data completeness, strict byte ordering, and reliable delivery are mandatory (financial, auth, document transfer).
* **Use UDP when**: Speed, ultra-low latency, and live timeliness matter more than 100% packet arrival (video calling, gaming, metrics ingestion).

---

### 5. Latency vs Throughput

**1. ELI5 Analogy**
* A highway water pipe: **Latency** is the time it takes for a single drop of water to travel from the reservoir to your kitchen tap (speed). **Throughput** is how many gallons of water pour out of the tap per minute (volume/capacity).

**2. Technical Explanation**
* **Latency** is the duration required for a single request or data packet to travel from sender to receiver and receive a response, typically measured in milliseconds (ms) across percentiles (p50, p95, p99). **Throughput** is the aggregate volume of units (requests, operations, bytes) processed by a system per unit of time, typically quantified as Requests Per Second (RPS) or Queries Per Second (QPS). While related through Little's Law ($L = \lambda W$), high throughput does not automatically guarantee low latency, and optimizing for batch throughput often degrades individual tail latency.

**3. Real-World Example**
* **High Throughput**: **Apache Kafka** batches thousands of events into disk writes, achieving millions of events/sec (huge throughput) at the expense of slight batching latency (~10-50ms).
* **Low Latency**: **Citadel / High-Frequency Trading (HFT)** engines optimize for microsecond execution of single trades, sacrificing massive concurrent batch capacity to guarantee sub-millisecond execution speeds.

**4. Tools & Alternatives**
* **Benchmarking Latency**: `wrk2`, `k6`, `vegeta` (stress test servers while measuring tail latency percentiles like p99/p99.9).
* **High-Throughput Systems**: Kafka, Hadoop MapReduce, Spark (optimized for parallel batch volume).
* **Low-Latency Systems**: Redis, Aerospike, C++ trading gateways (in-memory, zero-copy, kernel-bypass networking).

**5. When to use it / When NOT to**
* **Optimize for Latency when**: Building user-facing interactions (search typeahead, checkout click, voice assistant) where human perception degrades above 100ms.
* **Optimize for Throughput when**: Processing offline asynchronous workloads (ETL data pipelines, log aggregation, video transcoding) where total volume matters and delays of seconds or minutes are acceptable.

---

### One-Line Summary Recap (Topics 1–5)
> The **Client-Server Model** separates requester from provider, **DNS** resolves names to addresses, **HTTP/S** standardizes the application dialogue, **TCP vs UDP** trades reliability for raw speed at transport, and **Latency vs Throughput** balances single-turnaround time against total system volume.
