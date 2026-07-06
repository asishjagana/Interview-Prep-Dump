# Gap Analysis — 06_Caching_Performance

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 5 (3 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Caching_Strategies.md | Cache-aside, write-through, write-back, write-around |
| Cache_Eviction_Policies.md | LRU, LFU, FIFO, TTL, scan resistance |
| HTTP_Caching.md | Browser/client caching, Cache-Control, ETags, 304 validation |
| CDN.md | Edge PoPs, pull vs push, CDN invalidation, origin shield |
| GSLB.md | Global server load balancing, DNS/anycast routing, regional failover |
| Database_Connection_Pooling.md | Warm connection reuse, pool sizing |
| Data_Compression_Serialization.md | Binary schemas (Protobuf/Avro), compression trade-offs |
| Prefetching_Lazy_Loading.md | Eager vs lazy loading, N+1 mitigation |
| Performance_Metrics.md | P50/P99 latency, percentiles over averages |
| Profiling_Benchmarking.md | Bottleneck profiling, benchmarking methodology |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Distributed_Caching_Redis_Memcached.md** — Redis vs Memcached architecture, Redis Cluster vs client-side sharding, replication, persistence (RDB/AOF), data structures as cache primitives. (Redis appears only in passing inside `Caching_Strategies.md`; every real system-design answer names one of these two, and neither has a dedicated file.)
2. **Cache_Invalidation_Consistency.md** — TTL vs event-driven invalidation, cache-DB dual-write inconsistency, cache stampede / thundering herd mitigation (request coalescing, locking, probabilistic early expiration), stale-while-revalidate. (Stampede/thundering herd gets exactly one passing mention each in `Caching_Strategies.md` and `CDN.md` — the "hardest problem" the README itself names has no dedicated treatment.)
3. **Hot_Key_Problem_Multi_Tier_Caching.md** — Celebrity/hot-key skew, key splitting and replication, local (in-process) + remote (L1/L2) cache tiers, coherence between tiers. (Classic follow-up to any "cache it in Redis" answer; absent everywhere.)

### Optional / nice-to-have

4. **Cache_Warming_Strategies.md** — Pre-warming after deploy/failover/eviction flush, priming from access logs, cold-start surge protection. (Only passing "warm-up" mentions in `Profiling_Benchmarking.md` and elsewhere.)
5. **In_Process_vs_Distributed_Cache.md** — Caffeine/Guava-style local caches vs a shared cache cluster, near-cache pattern, serialization and network cost trade-offs. (Could alternatively be folded into #3.)

## Deliberately NOT flagged (covered in other folders)

- Consistent hashing for cache-node placement → `07_Load_Balancing_Scalability/Consistent_Hashing.md`
- Precomputation & materialized views (cache-adjacent read optimization) → `14_Performance_Optimization_Scalability/Precompute_Materialized_Views.md`
- Edge compute beyond static caching → `13_Cloud_Native_Infrastructure/Edge_Computing_Cloudfront_Functions.md`
- Database-side read optimization (indexing, replicas) → `05_Databases_Storage/Database_Indexing.md`, `05_Databases_Storage/Replication.md`
- Latency numbers underpinning cache decisions → `01_Planning_Foundations/Latency_Numbers_Every_Programmer_Should_Know.md`
- Golden signals / metrics pipelines → `12_Monitoring_Logging_Observability`

## Folder housekeeping

- **Duplicate file across folders:** `Database_Connection_Pooling.md` exists BOTH here and in `14_Performance_Optimization_Scalability/`. Pick one owner (pooling-as-caching-warm-connections fits here; 14 could cross-link instead of duplicating) and merge any unique content.
- **Overlap with 14:** `Data_Compression_Serialization.md` overlaps `14_Performance_Optimization_Scalability/Binary_Protocols_Protobuf_Avro.md`, and `Profiling_Benchmarking.md` overlaps `14_Performance_Optimization_Scalability/Performance_Testing_Benchmarking.md`. Not duplicates, but each pair should declare its angle and cross-link to avoid drift.
- `GSLB.md` is as much global traffic routing as caching; it lives here as the last hop of the edge chain, but it should cross-link prominently from `07_Load_Balancing_Scalability` (see that folder's audit — DNS/GeoDNS gaps there defer to this file).
