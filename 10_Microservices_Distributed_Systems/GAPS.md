# Gap Analysis — 10_Microservices_Distributed_Systems

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 10 (8 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Monolithic_vs_Microservices.md | Architecture comparison & migration drivers |
| Strangler_Fig_Pattern.md | Incremental migration |
| API_Gateway_Sidecar_Pattern.md | Edge & sidecar patterns |
| Service_Discovery_Consul_Eureka.md | Service registry & discovery |
| Service_Mesh_Istio_Linkerd.md | Mesh, mTLS, traffic management |
| Circuit_Breaker_Bulkhead.md | Resilience (breaker, bulkhead) |
| Orchestration_vs_Choreography.md | Workflow coordination styles |
| Distributed_Consensus_Raft_Paxos.md | Consensus algorithms |
| Distributed_Tracing_Jaeger.md | Cross-service tracing |
| Configuration_Management_Vault.md | Config & secrets distribution |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Saga_Pattern_Distributed_Transactions.md** — Choreographed vs orchestrated sagas, compensating transactions. (2PC/Saga touched in `05_Databases_Storage/Distributed_Transactions.md` from a DB angle, but the microservices-pattern treatment belongs here; cross-link the two.)
2. **CQRS_Pattern.md** — Command/query separation, read models, pairing with event sourcing. (Only passing mentions in 04/05/09 today.)
3. **Transactional_Outbox_Inbox_Pattern.md** — Dual-write problem, outbox + CDC relay, idempotent inbox consumers.
4. **Database_per_Service_Data_Ownership.md** — Data ownership boundaries, shared-DB anti-pattern, API composition vs materialized views for cross-service queries.
5. **Distributed_Locking_Leader_Election.md** — Redlock, ZooKeeper/etcd locks, fencing tokens, split-brain, leader election recipes.
6. **Logical_Clocks_Time_in_Distributed_Systems.md** — Lamport clocks, vector clocks, hybrid logical clocks, NTP/TrueTime. (Vector clocks only appear as side notes in 01/05.)
7. **Gossip_Protocol_Failure_Detection.md** — Gossip dissemination, heartbeats, phi-accrual, SWIM (used by Consul/Cassandra).
8. **Distributed_ID_Generation.md** — Twitter Snowflake IDs, UUID v4/v7, ticket servers, K-sortable IDs. (Classic interview staple; entirely absent from the repo.)

### Optional / nice-to-have

9. **Retry_Timeout_Backoff_Patterns.md** — Retries with jitter, timeout budgets, fallbacks, load shedding, retry storms. (Partially inside Circuit_Breaker_Bulkhead.md; split out if that file stays focused.)
10. **Backend_for_Frontend_BFF.md** — Per-client edge aggregation, vs a single shared API gateway.

## Deliberately NOT flagged (covered in other folders)

- Event Sourcing, exactly-once, DLQ/retry queues → `09_Message_Queues_Event_Streaming`
- Idempotency (API-level) → `02_API_Interface_Design/Idempotency.md`
- 2PC/3PC fundamentals → `05_Databases_Storage/Distributed_Transactions.md`
- Consistency models, quorum reads/writes → `05_Databases_Storage`
- DDD/bounded contexts → `04_Low_Level_Design/Domain_Driven_Design.md`
- Deployment strategies (blue-green, canary) → fits `13_Cloud_Native_Infrastructure` (flag in that folder's audit)

## Folder housekeeping

- **README.md is missing** — folders 01–03 and 05 have section indexes; add one here once gaps are filled.
