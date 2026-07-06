# Gap Analysis — 03_High_Level_Design

> Audit date: 2026-07-05. Existing files: 8. Missing (proposed): 6 (3 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Monolith_vs_Microservices.md | The style choice, Conway's Law, distributed monolith |
| Modular_Monolith.md | The "third way": enforced module boundaries in one deployable |
| SOA.md | ESB-centric services, contrast with microservices |
| Event_Driven_Architecture.md | Async events, decoupling, outbox/idempotency needs |
| Stateless_vs_Stateful_Services.md | Where state lives; externalizing to scale the app tier |
| System_Decomposition.md | Bounded contexts, splitting by capability not layer |
| Component_Responsibilities.md | SRP at architecture zoom, cohesion/coupling, IO at edges |
| High_Availability_Design.md | Redundancy, failover, availability arithmetic |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Layered_Hexagonal_Clean_Architecture.md** — The intra-service architecture styles: classic N-tier/layered (presentation–domain–data), hexagonal/ports-and-adapters, onion/clean, and when layering becomes lasagna. Every "draw the boxes inside one service" question lands here. (Today only a table row and a Staff+ nuance inside `Component_Responsibilities.md` plus side mentions in 04; deserves the dedicated treatment, cross-linked from both.)
2. **Multi_Tenancy_Architecture.md** — Tenancy models (silo / pool / bridge, shared-nothing vs shared-everything), tenant isolation levels, noisy-neighbor mitigation, per-tenant scaling and cost attribution, tenant-aware routing. A SaaS-design staple with no dedicated file — only side mentions in `02_API_Interface_Design/Authorization.md` (tenant scoping) and `05_Databases_Storage/Sharding_Partitioning.md` (shard by tenant_id); cross-link both.
3. **Lambda_Kappa_Big_Data_Architecture.md** — The big-data architecture styles: Lambda (batch + speed layers), Kappa (everything is a stream), and the modern lakehouse/streaming-first successors; when an analytics/aggregation design question calls for each. (Passing mentions only, in `09_Message_Queues_Event_Streaming/Stream_Processing.md` and `14_Performance_Optimization_Scalability/Batching_vs_Streaming.md`; the *architecture-style* decision belongs here.)

### Optional / nice-to-have

4. **Cell_Based_Architecture.md** — Cells as self-contained fault-isolation units (AWS, Slack, DoorDash), cell routing, blast-radius math, shuffle sharding. (Currently a single table row in `System_Decomposition.md`; strong Staff+ differentiator.)
5. **Micro_Frontends.md** — Extending service decomposition to the UI: composition strategies (build-time, runtime, edge-side), shared design systems, when it's overkill. Rounds out the decomposition story for full-stack interviews.
6. **Client_Server_vs_Peer_to_Peer.md** — The topology axis: client-server, P2P/hybrid (BitTorrent, WebRTC mesh vs SFU), gossip-style dissemination at the architecture level; shows up in file-sharing and live-video design questions.

## Deliberately NOT flagged (covered in other folders)

- Serverless/FaaS as an architecture style → `13_Cloud_Native_Infrastructure/Serverless_Computing_Lambda.md`
- CQRS and Saga (the EDA companion patterns) → proposed as core gaps in `10_Microservices_Distributed_Systems/GAPS.md` (`CQRS_Pattern.md`, `Saga_Pattern_Distributed_Transactions.md`)
- Event sourcing, message-queue mechanics → `09_Message_Queues_Event_Streaming`
- Strangler Fig migration → `10_Microservices_Distributed_Systems/Strangler_Fig_Pattern.md` (referenced from `System_Decomposition.md`)
- Sync vs async processing trade-off → `14_Performance_Optimization_Scalability/Async_vs_Sync_Processing.md`
- Batch vs streaming *processing* mechanics → `14_Performance_Optimization_Scalability/Batching_vs_Streaming.md` (the style-level Lambda/Kappa file above should link here)
- Active-active vs active-passive, health checks, failover mechanics → `07_Load_Balancing_Scalability`
- DDD/bounded-context depth → `04_Low_Level_Design/Domain_Driven_Design.md`
- Edge computing as a topology → `13_Cloud_Native_Infrastructure/Edge_Computing_Cloudfront_Functions.md`

## Folder housekeeping

- **Near-duplicate across folders** — `Monolith_vs_Microservices.md` (here) and `10_Microservices_Distributed_Systems/Monolithic_vs_Microservices.md` cover the same comparison under two nearly identical names. Keep this one as the style-choice treatment, refocus (or merge away) the 10 copy toward migration drivers, and cross-link; today neither references the other.
- **HA split across 03 and 07** — `High_Availability_Design.md` (here) vs `07_Load_Balancing_Scalability/High_Availability_Active_Active_vs_Active_Passive.md` and `Fault_Tolerance_vs_Resilience.md`. The design-level vs mechanism-level split is defensible, but state it explicitly in each file to stop future overlap.
