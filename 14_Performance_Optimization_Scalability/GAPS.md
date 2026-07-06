# Gap Analysis — 14_Performance_Optimization_Scalability

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 5 (3 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Latency_Throughput_Bandwidth.md | The three metrics and why they're not interchangeable |
| CPU_vs_Memory_vs_IO_Bound.md | Bottleneck diagnosis before scaling |
| Parallelism_vs_Concurrency.md | Structure vs hardware; cores vs interleaved waits |
| Async_vs_Sync_Processing.md | Non-blocking I/O, event loops, thread-per-request escape |
| Batching_vs_Streaming.md | Freshness vs efficiency, micro-batching |
| Precompute_Materialized_Views.md | Pay-once background computation, refresh policies |
| Binary_Protocols_Protobuf_Avro.md | Schema-typed binary wire formats vs JSON |
| Database_Connection_Pooling.md | Handshake reuse, pool sizing (see housekeeping) |
| Optimistic_vs_Pessimistic_Locking.md | Conflict-rate-driven locking choice |
| Performance_Testing_Benchmarking.md | Load/stress/soak/spike testing |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Garbage_Collection_Memory_Management.md** — Generational vs concurrent collectors (G1/ZGC/Shenandoah), stop-the-world pauses and their effect on tail latency, allocation pressure, off-heap memory, when to pick a runtime without GC. (Only passing "garbage collection" mentions exist repo-wide; GC pauses are the canonical answer to "why does p99 spike?")
2. **Tail_Latency_p99_Hedged_Requests.md** — Percentiles vs averages, tail amplification under fan-out (one slow shard slows every request), hedged/backup requests, timeout budgets, coordinated omission in benchmarks. (Percentiles are touched in `06_Caching_Performance/Performance_Metrics.md` and `12_Monitoring_Logging_Observability/Golden_Signals_Latency_Traffic_Errors.md`, but the tail-latency *engineering* toolkit — hedging, fan-out math — has no dedicated file.)
3. **Queueing_Theory_Littles_Law.md** — Little's Law (L = λW), the utilization–latency hockey stick, why 80% utilization is the practical ceiling, M/M/1 intuition for sizing pools/queues/headroom. (Little's Law appears inside `01_Planning_Foundations/QPS_TPS_Estimation.md` from an estimation angle — cross-link; the performance-modeling treatment that explains pool sizing and backpressure math belongs here.)

### Optional / nice-to-have

4. **Zero_Copy_IO_Optimizations.md** — Zero-copy (`sendfile`), mmap, page cache, io_uring, kernel bypass (DPDK); why Kafka and Nginx are fast. (Single passing mentions in 06/09/14 today; a strong Staff+ differentiator.)
5. **Thread_Pools_Sizing_Work_Queues.md** — Thread-pool sizing formulas (CPU-bound = cores, I/O-bound = cores × (1 + wait/compute)), bounded queues vs unbounded, rejection policies, thread-pool isolation per dependency. (Adjacent material in Parallelism_vs_Concurrency.md and 10's Circuit_Breaker_Bulkhead.md, but the sizing discipline itself is uncovered.)

## Deliberately NOT flagged (covered in other folders)

- Caching strategies, eviction, CDN, HTTP caching → `06_Caching_Performance`
- Profiling & benchmarking tools → `06_Caching_Performance/Profiling_Benchmarking.md` and `12_Monitoring_Logging_Observability/Profiling_Distributed_Profiling.md`
- Back-of-envelope estimation & latency numbers → `01_Planning_Foundations/Back_of_the_Envelope_Estimation.md`, `Latency_Numbers_Every_Programmer_Should_Know.md`
- Rate limiting & load shedding → `02_API_Interface_Design/Rate_Limiting.md` and `09_Message_Queues_Event_Streaming/Backpressure_Throttling.md`
- Auto-scaling & consistent hashing → `07_Load_Balancing_Scalability`
- Indexing & query-level DB performance → `05_Databases_Storage/Database_Indexing.md`
- Data compression & serialization (general) → `06_Caching_Performance/Data_Compression_Serialization.md`

## Folder housekeeping

- **Database_Connection_Pooling.md DUPLICATES `06_Caching_Performance/Database_Connection_Pooling.md`** — two independently written files (96 vs 113 lines) on the exact same topic, both indexed by their sections' READMEs. Pick one home (06 already groups it with its other resource-reuse topics) and replace the other with a cross-link, or merge the best of both into a single file.
- When gap files are added (or the duplicate is removed), update README's mermaid map, file index, and study paths (they currently hard-code "the 10 topics").
