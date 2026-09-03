# Batch 1: Foundations (Topics 1–5)

---

### 1. Client-Server Model

1. **ELI5 Analogy**: Think of a restaurant. You (the client) sit at a table and make an order. The kitchen (the server) receives your order, prepares the dish, and serves it back to you. You never cook the food yourself; you just ask and receive.
2. **Technical Explanation**: A distributed application architecture that partitions tasks between resource/service providers (servers) and service requesters (clients). Clients initiate communication sessions by sending requests over a network, while servers passively listen on designated ports, process inbound requests, and return responses. This model enables centralized data management, security enforcement, and business logic isolation away from end-user devices.
3. **Real-World Example**: **Web browsers (Chrome) & Google Search**. Your browser (client) sends search queries over the internet to Google's web server fleet, which parses queries, queries indices, and renders results back.
4. **Tools & Alternatives**:
   * **Client-Server**: Centralized authority (e.g., standard web/mobile apps using NGINX/Node.js backends).
   * **Peer-to-Peer (P2P)**: Decentralized model where nodes act as both client and server (e.g., BitTorrent, IPFS).
   * **Actor Model**: Message-driven decoupled concurrency model (e.g., Akka, Erlang/OTP).
5. **When to use it / When NOT to**:
   * **Use when**: You need centralized data control, auth, billing, and predictable access controls.
   * **Don't use when**: High fault tolerance without single points of failure is paramount or local direct node-to-node data transfer is cheaper/faster (e.g., decentralized torrenting).

---

### 2. DNS (Domain Name System)

1. **ELI5 Analogy**: DNS is the contact book on your phone. You don't memorize everyone's 10-digit phone number; you just tap "Mom" and your phone looks up the number behind the scenes to place the call.
2. **Technical Explanation**: A hierarchical, decentralized naming system that translates human-friendly hostnames (e.g., `api.example.com`) into machine-routable IP addresses (IPv4/IPv6). Resolution traverses local caches, recursive resolvers, root name servers, TLD (Top-Level Domain) servers, and authoritative name servers. Modern DNS also supports Anycast routing, geo-steering, and health-check-based failover.
3. **Real-World Example**: **Cloudflare DNS (1.1.1.1)** and **AWS Route 53**. Route 53 resolves traffic for companies like Airbnb and uses latency-based routing to direct users to the nearest AWS region.
4. **Tools & Alternatives**:
   * **AWS Route 53**: Managed public/private DNS with latency, weighted, and failover routing policies.
   * **Cloudflare DNS**: Ultra-fast authoritative and recursive resolver (1.1.1.1) with built-in DDoS protection.
   * **CoreDNS**: Lightweight, pluggable DNS server widely used for internal service discovery in Kubernetes clusters.
   * **BIND9**: The legacy open-source standard for self-hosted DNS infrastructure.
5. **When to use it / When NOT to**:
   * **Use when**: Directing public client traffic to public IPs, performing global multi-region routing, or internal Kubernetes service resolution.
   * **Don't use when**: Sub-millisecond real-time routing changes are required, because client and ISP DNS caches obey TTLs and won't update instantaneously.

---

### 3. HTTP/HTTPS & Request-Response Cycle

1. **ELI5 Analogy**: HTTP is sending a regular postcard where anyone handling the mail can read it. HTTPS is putting that message inside a locked metal case that only you and the recipient have keys to unlock.
2. **Technical Explanation**: HTTP (HyperText Transfer Protocol) is a stateless, application-layer protocol operating over TCP/IP using a request-response cycle consisting of methods (GET, POST, etc.), headers, status codes, and payloads. HTTPS wraps this transport in TLS/SSL encryption, enforcing confidentiality, integrity, and server authentication via cryptographic certificates. Modern standards (HTTP/2, HTTP/3) multiplex streams over a single connection to eliminate Head-of-Line blocking.
3. **Real-World Example**: **Stripe Checkout**. When a customer submits credit card info, the browser sends an HTTPS POST request with TLS 1.3 encryption so man-in-the-middle attackers cannot intercept payment credentials.
4. **Tools & Alternatives**:
   * **HTTP/1.1**: Simple, text-based, one request per TCP connection at a time (pipelining rarely supported).
   * **HTTP/2**: Binary framing, multiplexed streams over one TCP connection, header compression (HPACK).
   * **HTTP/3 (QUIC)**: Runs over UDP instead of TCP to eliminate transport-level Head-of-Line blocking and accelerate connection handshakes.
5. **When to use it / When NOT to**:
   * **Use when**: Standard client-server APIs, web apps, RESTful services, and document retrieval.
   * **Don't use when**: High-frequency bi-directional duplex communication is needed with minimal overhead (use WebSockets or raw TCP/UDP).

---

### 4. TCP vs UDP

1. **ELI5 Analogy**: TCP is registered certified mail: every delivery requires a return receipt signature; if a package is lost, it gets re-sent. UDP is a live TV broadcast or shouting across a field: you send the signals out continuously; if you miss one frame, you don't stop the show to go back.
2. **Technical Explanation**: Both are Layer 4 transport protocols. TCP (Transmission Control Protocol) is connection-oriented, requiring a 3-way handshake (`SYN`, `SYN-ACK`, `ACK`), guaranteeing in-order delivery, packet error-checking, retries, and congestion/flow control at the cost of latency. UDP (User Datagram Protocol) is connectionless and lightweight, sending datagrams with zero connection setup, no retransmissions, and no delivery guarantees, offering minimal latency.
3. **Real-World Example**: **Zoom & Discord** use UDP for live voice/video streaming where losing occasional audio packets is preferable to buffering delays; **GitHub & Banking APIs** use TCP to guarantee zero dropped or corrupted bytes in code commits or financial transactions.
4. **Tools & Alternatives**:
   * **TCP**: Built-in OS transport used by HTTP/1.1, HTTP/2, SSH, SMTP, and databases.
   * **UDP**: Built-in OS transport used by DNS lookups, VoIP, live streaming, and multiplayer online gaming.
   * **QUIC**: Google-designed hybrid that runs on UDP while implementing reliability and encryption at the application layer.
   * **SCTP**: Message-oriented transport supporting multi-homing and multi-streaming, common in telecom.
5. **When to use it / When NOT to**:
   * **Use TCP when**: Data integrity and complete, ordered delivery are strictly non-negotiable (files, financial transactions, web APIs).
   * **Use UDP when**: Real-time speed and low latency matter far more than dropped packets (audio/video calls, gaming, real-time telemetry).

---

### 5. Latency vs Throughput

1. **ELI5 Analogy**: Imagine a highway. Latency is the speed limit (how many minutes it takes for one car to travel from town A to town B). Throughput is the number of lanes (how many cars cross the finish line per hour).
2. **Technical Explanation**: Latency is the time delay required for a single data packet or request to travel from source to destination and return (measured in milliseconds, `ms`). Throughput is the volume of data or number of operations successfully processed per unit of time (measured in queries per second `QPS`, requests per second `RPS`, or `Gbps`). A system can have high latency and high throughput (e.g., a cargo freight train carrying petabytes of hard drives), or low latency and low throughput.
3. **Real-World Example**: **High-Frequency Trading (Jane Street)** optimizes for ultra-low single-digit microsecond latency; **Netflix Batch Video Encoding** optimizes for raw throughput (processing gigabytes per second), where whether a video takes 10 or 12 minutes to finish rendering is irrelevant.
4. **Tools & Alternatives**:
   * **Latency Optimization**: CDNs (Cloudflare), in-memory caches (Redis), local edge runtimes (Cloudflare Workers), connection keep-alive.
   * **Throughput Optimization**: Horizontal worker scaling, batching/pipelining (Kafka), asynchronous event queues, connection pooling.
   * **Benchmarking Tools**: `wrk`, `k6`, `Apache JMeter` (to measure p50/p95/p99 latency under target throughput loads).
5. **When to use it / When NOT to**:
   * **Prioritize Latency when**: Interactive user-facing experiences (search suggestions, checkout flows, gaming, voice communication).
   * **Prioritize Throughput when**: Offline processing, ETL pipelines, large-scale data ingestion, and batch analytics jobs.

---

### 💡 Quick Review Recap (Topics 1–5)
> **Recap**: The **Client-Server model** establishes who asks and who serves; **DNS** maps names to addresses; **HTTP/HTTPS** defines the secure communication rules; **TCP/UDP** governs reliable vs fast delivery at the transport layer; and **Latency vs Throughput** measures how fast one request completes vs how much work the system handles simultaneously.
