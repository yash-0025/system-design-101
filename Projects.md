# System Design — Project Practice Prompt (Part 2, v2 — Explicit Only, No AI Discretion)

Use this AFTER you finish all 115 topics from `system-design-fast-batch-prompt-v2.md`. Paste this into the same AI conversation (so it remembers the topics you covered) or a fresh one.

---

## THE PROMPT (copy everything below)

I've now learned the core system design concepts, tools, and terminology from a fixed 115-topic roadmap (databases, caching, load balancing, messaging, consistency, scalability patterns, payments, security, observability, deployment, and explicit real-world failure patterns like retry storms, split-brain, silent data corruption, vendor outages, etc.). Now I want to apply everything through **5 complete, hands-on system design projects** — 1 small, 2 medium, and 2 big.

**Important constraint: do not invent or introduce concepts, scenarios, or terminology beyond what is explicitly listed below in this prompt.** I want to minimize hallucination risk, so every scenario, failure mode, and concept you must cover is explicitly named in this document already — your job is to explain and apply each of these listed items in depth for each project, not to generate new ones on your own judgment. If something listed doesn't apply to a given project, say so explicitly and explain why, rather than substituting something unlisted.

I will be drawing every diagram myself in **Eraser.io**, so you must also give me literal, ordered, step-by-step build instructions at the end of each project (e.g., "Step 1: Draw a box labeled 'Client (Web/Mobile)'. Step 2: Draw a box labeled 'Load Balancer' below it and connect Client → Load Balancer with an arrow labeled 'HTTPS Request'.") so I can follow along and build the diagram box-by-box and arrow-by-arrow without having to interpret or reorganize anything myself.

### Finalized Project Roadmap (go in this exact order, don't suggest alternatives — use these)

**Small**
1. **URL Shortener**

**Medium**
2. **E-Commerce Platform**
3. **Real-Time Chat / Messaging App (WhatsApp-style)**

**Big**
4. **Live Streaming Platform (Twitch/YouTube Live-style) with Adaptive Upscaling Architecture**
5. **High-Frequency Trading (HFT) System**

Go through them strictly in this order: Small → Medium (E-Commerce) → Medium (Chat App) → Big (Live Streaming) → Big (HFT). Do ONE project fully before moving to the next.

### For EACH of the 5 projects, cover ALL of the following sections IN FULL DETAIL:

**1. Problem Statement**

**2. Requirements Gathering** — functional requirements, non-functional requirements (scale, latency, availability target, consistency needs), clarifying questions a real engineer would ask.

**3. Capacity Estimation** — full worked math: users, read/write QPS (peak and average), storage growth, bandwidth, memory/cache sizing.

**4. High-Level Architecture** — every component named and explained, full explicit data flow path, logical layer grouping (Client, Edge/CDN, Load Balancing, Application/Service, Data, Async/Messaging, Observability).

**5. Scaling Strategy** — vertical vs horizontal for every component and why, load balancer algorithm choice, autoscaling triggers, database sharding/replication approach, cache scaling, and exactly what changes at 1K → 1M → 100M users (or low → peak volume for HFT).

**6. Caching Strategy** — exactly what's cached, where, which pattern (cache-aside/write-through/write-back), TTLs, invalidation triggers, cache stampede prevention.

**7. Deep Dive on Each Component** — what it does here, exact tool chosen, why over the alternatives.

**8. Data Model / Schema** — key tables/structures with relationships.

**9. Idempotency & Consistency** — every operation that must be idempotent and how it's enforced; consistency model per critical data path (strong vs eventual) with justification.

**10. Payments & Financial Flow Handling** (apply if the project involves money movement — E-Commerce and HFT; for other projects, state "not applicable" and move on):
- Payment Idempotency & Duplicate Charge Prevention
- Payment Gateway Timeout/Retry Handling & Webhook Reconciliation
- Third-Party/Vendor Outage Fallback Strategies

**11. Error Handling & Failure Scenarios** — walk through: DB down, payment gateway timeout, server crash mid-request, message queue backup, network partition between regions. For each: exact mechanism (retries/backoff, circuit breakers, DLQs, fallbacks, graceful degradation, timeouts), how it's surfaced, and how reconciliation happens after recovery.

**12. Explicit Real-World Failure Pattern Checklist** — go through this exact fixed list for this project. For each item: explain whether/how it applies to THIS specific project, and if it applies, explain the concrete mitigation used in this architecture. If not applicable, say so briefly and move on — do not skip silently, and do not add items not on this list.
- Flash Crowd / Traffic Spike Handling (viral events, flash sales, big live events)
- Cache Stampede / Thundering Herd Problem
- Cascading Failure Prevention (one slow service taking down others)
- Retry Storms & Client-Side Backoff
- Duplicate Event/Message Processing & Deduplication Strategies
- Clock Skew & Distributed Event Ordering
- Hot Keys / Hot Partitions Problem
- Split-Brain Scenarios (leader election / network partition)
- Zero-Downtime Database Migration & Schema Changes on Live Systems
- Bad Deploy Recovery — Instant Rollback & Traffic Draining
- Third-Party/Vendor Outage Fallback Strategies (if not already covered in section 10)
- Bot Traffic, Scraping & Abuse/Rate-Limit Evasion
- Silent Data Corruption Detection (checksums, reconciliation jobs, partial write handling)
- Dead Letter Queues & Backpressure Handling
- Credential Stuffing & Replay Attack Mitigation

**13. Trade-offs Made** — table of "we chose X over Y because Z" for the 3-5 biggest decisions.

**14. Step-by-Step Eraser.io Diagram Blueprint** — literal numbered build sequence ("Step 1: Create box labeled '___'. Step 2: Create box labeled '___', connect with arrow labeled '___'.") grouped by layer, top-to-bottom, every arrow labeled with what flows on it and sync/async, and which boxes should be visually grouped together.

### Rules for how we go through this

- Do ONE project fully, covering every numbered section above including the full checklist in section 12 (item by item, not skipped or summarized), before moving to the next.
- Stick strictly to the concepts and scenarios named in this prompt — no invented terminology or scenarios.
- Depth over brevity — split a project across multiple responses if needed rather than compressing sections.
- Use tables and bullet points over long paragraphs wherever possible, but never sacrifice technical depth for brevity.
- Mention what a system design interviewer would specifically probe on in that project.
- After each project, give me a short recap (5-6 bullet points) of the key decisions and which checklist items from section 12 applied, then ask if I'm ready to move to the next project.

---

**Start with the Small project: URL Shortener. Cover every section in full detail, including the full section 12 checklist item by item.**