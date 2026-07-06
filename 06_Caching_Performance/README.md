# 06 · Caching & Performance ⚡

> **The read-scaling layer that sits in front of the database.** Every cache — a browser's local store, a CDN edge node, a Redis cluster — is a bet that recently-used data will be used again soon. This section is that bet made explicit: which caching strategy and eviction policy to run, how to push content all the way to the edge, how to cut the cost of the requests that still have to travel, and how to prove any of it actually worked. Every decision here trades staleness or CPU for speed; get the trade wrong and you've either served garbage fast or served the truth too slowly.

---

## 🎯 What You'll Be Able To Do

- **Pick a caching strategy from the access pattern** — cache-aside, write-through, write-around, write-back — and defend the staleness/durability/speed trade-off it makes.
- **Choose an eviction policy deliberately** — LRU, LFU, FIFO, TTL — and know which one survives a one-time scan without getting polluted.
- **Push content to the edge** — HTTP cache headers, ETags, and CDN PoPs — so most requests never reach your origin server at all.
- **Route users globally and survive a regional outage** — GSLB, health checks, and failover without triggering a cascading collapse.
- **Cut the cost of every request that isn't a cache hit** — connection pooling, binary serialization, compression, and prefetch/lazy-load.
- **Prove it got faster** — read P50/P99 latency correctly, profile the real bottleneck, and benchmark before claiming a win.

---

## 🗺️ How the 10 Topics Interlock

```mermaid
flowchart TD
    CACHE["Caching layer — what to keep &amp; what to evict?"]
    CACHE --> CS["① Caching Strategies"]
    CACHE --> CE["② Cache Eviction Policies"]

    EDGE["Edge &amp; network caching — get it closer to the user"]
    EDGE --> HC["③ HTTP Caching"]
    EDGE --> CDN["④ CDN"]
    EDGE --> GSLB["⑤ GSLB"]

    EFFICIENCY["Resource efficiency — do less work per request"]
    EFFICIENCY --> POOL["⑥ DB Connection Pooling"]
    EFFICIENCY --> COMP["⑦ Compression &amp; Serialization"]
    EFFICIENCY --> PRE["⑧ Prefetching &amp; Lazy Loading"]

    MEASURE["Measurement — did it actually get faster?"]
    MEASURE --> PM["⑨ Performance Metrics"]
    MEASURE --> PB["⑩ Profiling &amp; Benchmarking"]

    CACHE --> EDGE --> EFFICIENCY --> MEASURE

    style CACHE fill:#ede9fe,color:#000
    style EDGE fill:#cffafe,color:#000
    style EFFICIENCY fill:#dcfce7,color:#000
    style MEASURE fill:#fef3c7,color:#000
```

> **The spine:** *pick a caching strategy and an eviction policy → push that same idea outward to the edge (browser → CDN → GSLB) → shrink the cost of every request that still has to travel → measure whether any of it actually worked.* Eviction (②) is what makes strategy (①) survive finite RAM; HTTP caching (③) and CDNs (④) are cache-aside applied at the edge instead of in Redis; GSLB (⑤) is the last hop of that same idea — routing around a slow or dead region. Pooling (⑥), compression (⑦), and prefetching (⑧) all cut the cost of the requests that don't get a cache hit; metrics (⑨) and profiling (⑩) are how you find out whether any of it moved the needle.

---

## 📚 The Complete File Index

| # | Topic | One-line | Key takeaway |
|---|-------|----------|--------------|
| 1 | [Caching Strategies](./Caching_Strategies.md) | The book on your desk | Cache-aside is the default; write-through buys integrity, write-back buys speed at the risk of loss; invalidation is the hard part |
| 2 | [Cache Eviction Policies](./Cache_Eviction_Policies.md) | Managing a small bookshelf | LRU fits most temporal-locality access patterns by default; LFU protects long-term popular items but suffers cold-start |
| 3 | [HTTP Caching](./HTTP_Caching.md) | A grocery list vs. its photo | Hash the filename + `max-age=1yr, immutable` for the perfect cache; ETags validate the rest; never mark private data `public` |
| 4 | [CDN](./CDN.md) | Corner store vs. the warehouse | Pull model + edge PoPs beat the speed of light; invalidation across hundreds of nodes is the recurring pain, never cache PII there |
| 5 | [GSLB](./GSLB.md) | A travel agent for the internet | DNS/anycast routes to the nearest healthy region; size failover for the surge or it cascades region to region |
| 6 | [Database Connection Pooling](./Database_Connection_Pooling.md) | The DMV with dedicated clerks | Reuse warm TCP/TLS connections instead of paying a 50-100ms handshake per request; pool size regulates load on the DB |
| 7 | [Data Compression & Serialization](./Data_Compression_Serialization.md) | Flat-pack furniture, then vacuum-sealed | Binary schemas (Protobuf/Avro) beat JSON for service-to-service traffic; compression trades CPU for bandwidth — measure the level |
| 8 | [Prefetching & Lazy Loading](./Prefetching_Lazy_Loading.md) | Order from the menu, refill early | Lazy-load what might never be needed, eager-load/prefetch what will be — the fix for the N+1 nightmare |
| 9 | [Performance Metrics](./Performance_Metrics.md) | The car's dashboard | Never trust the average — P99 is the unluckiest user's experience, and it's what makes a page feel broken |
| 10 | [Profiling & Benchmarking](./Profiling_Benchmarking.md) | Heart monitor, then the Olympic dash | Profile to find the real bottleneck, benchmark to prove the fix worked; sample in production, trace only locally |

---

## 🧭 Suggested Study Paths

```mermaid
flowchart LR
    subgraph FAST["⚡ Interview Crunch"]
        F1["1 Caching Strategies"] --> F2["2 Eviction"] --> F3["4 CDN"] --> F4["6 Pooling"] --> F5["9 Metrics"]
    end
    subgraph DEEP["🏗️ Mastery (full order)"]
        D1["1→2"] --> D2["3→4→5"] --> D3["6→7→8"] --> D4["9→10"]
    end
    style FAST fill:#fef3c7,color:#000
    style DEEP fill:#dcfce7,color:#000
```

| Goal | Path | Why |
|------|------|-----|
| **⚡ Interview in 48h** | 1 → 2 → 4 → 6 → 9 | Cache pattern + eviction + CDN + pooling + P99 latency — the most-asked 80% |
| **🏗️ Deep mastery** | 1→2 → 3→4→5 → 6→7→8 → 9→10 | Cache layer → push it to the edge → cut per-request cost → measure, in dependency order |
| **🌍 Global-scale focus** | 3 → 4 → 5 | Browser cache → CDN edge → GSLB failover: the full path content takes to reach a user anywhere |
| **📉 "Why is it slow?" diagnosis** | 9 → 10 → 1 → 2 | Measure first (find the P99 outlier), profile the bottleneck, then fix it with the right cache/eviction pattern |

> **Study each file by** attempting the 🏋️ **Practice Drills before the solutions**, rehearsing the 🎤 **Interview Q&A out loud**, and reading 🎯 **Staff+ Nuances** for senior-level depth.

---

## 🔑 The Decision Card (whole-section cheat sheet)

```
CACHE       cache-aside (DEFAULT, lazy) · write-through (integrity, slower writes) · write-back (fast, risk of loss) ·
            write-around (skip pollution) · cache invalidation is the hardest problem in CS
EVICT       LRU (DEFAULT, temporal locality) · LFU (long-term popularity, but cold-start hurts new items) ·
            FIFO/random (cheap, dumb) · TTL = expiration, not eviction · segment hot/cold to survive a one-time scan
HTTP        Cache-Control max-age + immutable on hashed filenames = the perfect cache · ETag/If-None-Match = 304 validation ·
            public (CDN/proxy may cache) vs private (browser only) — never serve PII as public
CDN         pull model + edge PoPs = workaround for the speed of light · push for big launches ·
            invalidation lags across hundreds of nodes · origin shield to survive a thundering-herd cache miss
GSLB        DNS/anycast routes to the nearest HEALTHY region · size failover for 2x load or it cascades region to region ·
            multi-region correctness is a replication/consistency problem, never free
EFFICIENCY  pool connections — reuse beats a handshake per request · binary serialization (Protobuf/Avro) > JSON service-to-service ·
            compression trades CPU for bandwidth, measure the level · prefetch/eager-load kills N+1, lazy-load defers the unneeded
MEASURE     NEVER trust the average — P99/P999 is the unlucky user's real experience · Golden Signals: latency, traffic, errors, saturation ·
            profile (sampling in prod, tracing locally) to find the bottleneck, benchmark to prove the fix actually helped
RECURRING   every cache is a bet that recently-used data gets reused again; every layer here trades STALENESS or CPU for SPEED
```

---

## 🎨 Conventions & Authoring

- Every file follows the root [TEMPLATE.md](../TEMPLATE.md). Animated SVGs live in [/assets](../assets/).
- Each file ends with the three mastery sections: 🏋️ Practice Drills, 🎤 Interview Q&A Bank, 🎯 Staff+ Nuances.

---

## ➡️ Where This Connects

- ⬅️ [01 · Planning & Foundations](../01_Planning_Foundations/) — latency numbers (the "RAM vs disk vs network" figures every caching decision is built on).
- ⬅️ [05 · Databases & Storage](../05_Databases_Storage/) — the layer being cached; every cache is a bet against re-hitting this tier.
- ➡️ [07 · Load Balancing & Scalability](../07_Load_Balancing_Scalability/) — routing and horizontal scale for the requests that still get through.
- ➡️ [14 · Performance Optimization & Scalability](../14_Performance_Optimization_Scalability/) — the broader optimization playbook this section's metrics and profiling feed into.

---
*Part of the [System Design & Architecture Master Repository](../README.md). This is the layer that decides whether "fast" or "correct-but-slow" is the default experience users get.*
