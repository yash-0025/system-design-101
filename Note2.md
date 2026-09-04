# Batch 2: Topics 6–10 (Foundations & Networking)

---

### 6. Availability vs Reliability vs Consistency

**1. ELI5 Analogy**
* **Availability**: An ATM that is always unlocked and ready to take your card 24/7.
* **Reliability**: When you withdraw $100, the ATM spits out exactly five crisp $20 bills without jamming or crashing, every single time.
* **Consistency**: If you withdraw $100 and immediately check your balance on your phone app, both the ATM receipt and the app show the exact same new balance instantly.

**2. Technical Explanation**
* Availability measures the percentage of time a service remains operational and answers requests successfully (e.g., "four nines" or 99.99% uptime), regardless of temporary data staleness. Reliability (often quantified via Mean Time Between Failures / MTBF) measures the probability that a system performs its intended function without error, crash, or data corruption over a given time interval. Consistency guarantees that every read operation receives the most recent write or an explicit error, ensuring all distributed nodes reflect the exact same state at any given point. While a system can be highly available by returning stale or degraded data, doing so directly trades off strict consistency and reliability guarantees.

**3. Real-World Example**
* **Availability**: **Amazon Storefront** prioritizes availability so shoppers can always add items to their cart, tolerating temporary stock inconsistency over turning away buyers with error screens.
* **Reliability & Consistency**: **Visa / SWIFT** prioritize strict consistency and transactional reliability; failing a transaction or making the user wait is preferred over processing an inconsistent balance or phantom payment.

**4. Tools & Alternatives**
* **High Availability (AP) Datastores (e.g., Apache Cassandra, DynamoDB)**: Multi-master architecture optimized for zero downtime and write availability with eventual consistency.
* **Strictly Consistent (CP) Datastores (e.g., CockroachDB, Google Spanner)**: Raft/Paxos-based distributed consensus systems guaranteeing serializable ACID transactions across regions.
* **Reliability Frameworks (e.g., Chaos Mesh, Gremlin)**: Chaos engineering tools to proactively inject faults and measure mean-time-to-recovery (MTTR).

**5. When to use it / When NOT to**
* **Prioritize Availability when**: Downtime directly kills user retention and stale data is safe to reconcile asynchronously (social feeds, product catalogs).
* **Prioritize Consistency & Reliability when**: Financial transactions, inventory counts, or identity management require absolute correctness where stale reads cause real-world loss.

---

### 7. Vertical Scaling vs Horizontal Scaling

**1. ELI5 Analogy**
* **Vertical Scaling (Scale Up)**: Upgrading a delivery truck with a massive monster-truck engine and reinforced bed so it can carry twice as many packages.
* **Horizontal Scaling (Scale Out)**: Buying a fleet of 10 standard delivery vans and distributing packages among 10 drivers.

**2. Technical Explanation**
* Vertical scaling (scaling up) involves adding more hardware resources—such as CPU cores, RAM, and faster NVMe storage—to a single existing node. Horizontal scaling (scaling out) involves provisioning additional independent nodes into a distributed pool, sharing incoming request traffic across them via a load balancer. While vertical scaling avoids architectural complexity, distributed data synchronization, and network latency, it hits hard physical hardware limits and represents a single point of failure (SPOF). Horizontal scaling provides near-linear capacity expansion and fault isolation, but introduces network latency, distributed state management, and orchestration overhead.

**3. Real-World Example**
* **Vertical Scaling**: **Stack Overflow** famously ran for years on a handful of massive multi-core, high-RAM on-premise MS SQL servers to avoid distributed database complexity.
* **Horizontal Scaling**: **Netflix** dynamically scales out thousands of stateless microservice container instances on AWS EC2 during prime-time evening hours and scales back in overnight.

**4. Tools & Alternatives**
* **Vertical Scaling Enablers (AWS EC2 bare-metal, `u-24tb1.metal`)**: Single cloud instances scaling up to hundreds of vCPUs and terabytes of RAM.
* **Horizontal Orchestrators (Kubernetes, AWS ECS, Nomad)**: Container schedulers that automatically spin up or tear down task replicas based on CPU/RAM/traffic metrics.
* **Auto-Scalers (KEDA, Horizontal Pod Autoscaler)**: Event-driven autoscaling tools that scale pods horizontally based on queue depth or Prometheus metrics.

**5. When to use it / When NOT to**
* **Use Vertical Scaling when**: Traffic is predictable, simplicity is critical, and the workload fits comfortably on standard single-box hardware without distributed engineering overhead.
* **Use Horizontal Scaling when**: Workloads have volatile traffic spikes, require high availability with zero single-point-of-failure, or outgrow the limits of single-node hardware.

---

### 8. Back-of-the-Envelope Estimation (QPS, Storage, Bandwidth Math)

**1. ELI5 Analogy**
* Planning catering for a wedding: Instead of counting every crumb, you calculate: 200 guests × 3 drinks per person = 600 drinks; each bottle holds 5 drinks = 120 bottles needed, plus 20% extra for safety.

**2. Technical Explanation**
* Back-of-the-envelope estimation is the quantitative engineering practice of using ballpark figures, powers-of-ten approximations, and standard hardware constants to size compute, network bandwidth, memory, and disk storage requirements. Key metrics include Daily Active Users (DAU), read/write ratios, peak vs. average Queries Per Second ($\text{QPS} = \frac{\text{Total Requests}}{86,400\text{ sec}}$), ingress/egress bandwidth ($\text{QPS} \times \text{payload size}$), and multi-year storage accumulation ($\text{Daily writes} \times \text{size} \times 365 \times \text{retention years}$). These numbers directly dictate whether an architecture requires in-memory caching, database sharding, or multi-terabit network backbones.

**3. Real-World Example**
* **Twitter / X Design**: With 300M DAU posting 2 tweets/day each, write QPS is $\approx \frac{600\text{M}}{86,400} \approx 7,000 \text{ QPS}$ (peaking at $15,000\text{ QPS}$); with a 100:1 read/write ratio, read QPS reaches $700,000\text{ QPS}$, proving that in-memory timeline caches (Redis) and fan-out architectures are non-negotiable.

**4. Tools & Alternatives**
* **Mental Rules of Thumb**: $10^5$ seconds per day ($86,400 \approx 100,000$); $1 \text{ GB} = 10^9 \text{ bytes}$; RAM latency $\sim 100\text{ns}$, NVMe SSD read $\sim 100\mu\text{s}$, cross-datacenter RTT $\sim 50\text{ms}$.
* **Fermat / Calculation Tools**: Spreadsheet models, Python scratch scripts, or CLI tools (`bc`, `ipython`) for system capacity planning.

**5. When to use it / When NOT to**
* **Use when**: Initiating system design interviews, planning infrastructure budgets, deciding between monolithic vs. distributed designs, or vetting architecture feasibility.
* **Do NOT use when**: Sizing hard production SLA limits, financial billing, or memory bounds where precise profiling, load testing, and telemetry are required.

---

### 9. Load Balancers (L4 vs L7)

**1. ELI5 Analogy**
* **L4 Load Balancer**: A highway traffic cop waving cars into different highway lanes based purely on vehicle size or license plate, without looking inside the trunk.
* **L7 Load Balancer**: A concierge at a luxury hotel who opens your envelope, reads what department you need (dining, spa, or rooms), and walks you directly to that specific floor.

**2. Technical Explanation**
* A Layer 4 (L4) load balancer operates at the Transport layer (TCP/UDP), making routing decisions based purely on source/destination IP addresses and port numbers without inspecting application payload data. A Layer 7 (L7) load balancer operates at the Application layer, terminating the TLS/TCP connection and parsing HTTP/HTTPS headers, cookies, URL paths, and query parameters to route requests intelligently. L4 offers blistering packet throughput, minimal CPU utilization, and ultra-low latency, while L7 provides sophisticated capabilities like path-based routing (`/api` vs `/static`), SSL termination, sticky sessions, and HTTP header rewriting at the cost of higher CPU overhead.

**3. Real-World Example**
* **Netflix / AWS**: Uses an L4 load balancer (AWS NLB) for ultra-high-throughput streaming video ingestion and low-latency gaming streams, fronted by an L7 load balancer (AWS ALB or Envoy) to route microservice HTTP traffic based on URL paths (`/watch`, `/account`, `/recommendations`).

**4. Tools & Alternatives**
* **HAProxy**: Ultra-fast, battle-tested reverse proxy and load balancer supporting both L4 TCP proxying and L7 HTTP routing.
* **NGINX**: Premier L7 web server and reverse proxy with rich HTTP routing, caching, and SSL termination.
* **Envoy**: Modern, C++ high-performance cloud-native L7 proxy designed for microservices and service meshes with dynamic configuration.
* **AWS NLB (L4) vs AWS ALB (L7)**: Managed cloud standards; NLB handles millions of requests/sec with ultra-low latency; ALB provides path/host-based routing.

**5. When to use it / When NOT to**
* **Use L4 when**: Raw throughput, millisecond latency, protocol independence (TCP/UDP, gaming, VoIP), and packet-level balancing are top priority.
* **Use L7 when**: You need microservice path routing (`/users` vs `/orders`), cookie-based sticky sessions, header inspection, or SSL offloading.

---

### 10. Load Balancing Algorithms (Round Robin, Least Connections, Consistent Hashing)

**1. ELI5 Analogy**
* **Round Robin**: Dealing cards to players around a table: one card to player 1, one to player 2, one to player 3, and repeat.
* **Least Connections**: A grocery store manager directing the next shopper in line to whichever cashier has the shortest checkout line.
* **Consistent Hashing**: A neighborhood mail room assigning all mail for a specific apartment number to the exact same locker every day, so adding or removing locker banks doesn't scramble everyone's keys.

**2. Technical Explanation**
* Load balancing algorithms determine how incoming traffic is distributed across backend server pools. **Round Robin** cycles through nodes sequentially (optionally weighted by server capacity), assuming all requests have equal processing cost. **Least Connections** directs traffic to the server with the fewest active open connections, making it ideal for sessions with varying transaction times. **Consistent Hashing** maps request keys (e.g., user ID, client IP) and server nodes onto a circular hash ring ($0 \text{ to } 2^{32}-1$), ensuring that adding or removing a backend node only reshuffles $\frac{K}{N}$ keys rather than invalidating the entire cluster's cached state.

**3. Real-World Example**
* **Consistent Hashing**: **Discord** uses consistent hashing to assign voice channel sessions and cache data to specific servers, ensuring that if a server dies, only a tiny fraction of users are redistributed.
* **Least Connections**: **Database Connection Pools & WebSockets (e.g., Slack)** use least connections because WebSocket connections stay open for hours, making round robin disastrous if one server accumulates all long-lived connections.

**4. Tools & Alternatives**
* **Round Robin / Weighted Round Robin**: Default in NGINX, HAProxy, and DNS round-robin.
* **Least Connections / Least Response Time**: Standard in HAProxy and Envoy for heterogeneous request workloads.
* **Consistent Hashing (Ketama, Maglev)**: Implemented in Envoy, HAProxy, Memcached clients, and Google’s Maglev load balancer for stateful cache affinity.
* **Randomized / Power of Two Choices**: Picks two nodes at random and selects the one with fewer connections, avoiding thundering herds.

**5. When to use it / When NOT to**
* **Use Round Robin when**: Requests are uniform, stateless, and execute in similar timeframes.
* **Use Least Connections when**: Requests have wildly uneven durations (e.g., long file uploads, persistent streaming).
* **Use Consistent Hashing when**: You need cache affinity or session stickiness without relying on centralized session stores.

---

### One-Line Summary Recap (Topics 6–10)
> **Availability, Reliability & Consistency** define system uptime vs. functional correctness vs. data truth, **Vertical vs. Horizontal Scaling** balances single-box simplicity against distributed resilience, **Back-of-the-Envelope Estimation** quantifies capacity math before building, **L4 vs. L7 Load Balancers** choose between raw packet speed and intelligent application routing, and **Load Balancing Algorithms** dictate how traffic is divided across the fleet.
