# Gap Analysis — 01_Planning_Foundations

> Audit date: 2026-07-05. Existing files: 12. Missing (proposed): 4 (2 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| System_Design_Interview_Overview.md | The 5-phase interview process, time management |
| Requirement_Gathering.md | Functional vs non-functional vs constraints (the 3 buckets) |
| Constraints_Assumptions.md | Stating constraints & assumptions explicitly |
| Back_of_the_Envelope_Estimation.md | Order-of-magnitude math: DAU → QPS → storage → servers |
| Latency_Numbers_Every_Programmer_Should_Know.md | The fundamental latency constants |
| QPS_TPS_Estimation.md | Throughput, peak factor, Little's Law, the latency "knee" |
| Capacity_Planning.md | Load → server count, 4 resource dimensions, headroom, N+1 |
| Cost_Estimation_FinOps.md | Cloud cost math, egress, $/mo on every box |
| SLA_SLO_SLI.md | Reliability targets & error budgets |
| Availability_Reliability_Math.md | Series/parallel availability, MTBF/MTTR, the nines |
| CAP_Theorem_PACELC.md | Consistency vs availability (and latency) trade-off framing |
| Trade_off_Analysis.md | Naming the sacrifice; structured trade-off reasoning |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Scalability_Fundamentals_Scale_Cube.md** — What scalability actually means (scalability ≠ performance), vertical vs horizontal scaling as a *design* choice, the AKF Scale Cube (X = clones, Y = functional split, Z = data shards), diminishing returns of scale-up, and how to answer "how would you scale this 10×/100×?". (`07_Load_Balancing_Scalability/Auto_scaling_Horizontal_vs_Vertical.md` covers the autoscaling *mechanism*, not the conceptual scaling-dimensions framing an interviewer probes in the planning phase; cross-link the two.)
2. **Bottleneck_Analysis.md** — Systematically finding the limiting component: walk the request path, USE method (utilization/saturation/errors), read vs write amplification, fan-out multipliers, and "what breaks first at 10×?". (Fragments exist today — the "knee" and Little's Law in `QPS_TPS_Estimation.md`, the invisible-bottleneck failure modes in `Capacity_Planning.md` §10 — but the interviewer's favorite follow-up deserves its own method file; cross-link `09_Message_Queues_Event_Streaming/Backpressure_Throttling.md` for what happens when the bottleneck pushes back.)

### Optional / nice-to-have

3. **Interview_Communication_Framework.md** — The meta-skill beyond the 5 phases: thinking out loud, taking hints gracefully, disagreeing with the interviewer, whiteboard/Excalidraw hygiene, seniority signals (what L4 vs L6 answers sound like), and recovering when stuck. (Partially inside `System_Design_Interview_Overview.md` ("drive, but stay steerable"); split out if that file stays focused on process.)
4. **Architecture_Decision_Records.md** — ADRs as the written form of trade-off analysis: context → decision → consequences, when to write one, lightweight formats. Natural companion to `Trade_off_Analysis.md` and a common Staff+ discussion point.

## Deliberately NOT flagged (covered in other folders)

- Functional vs non-functional requirements → substantially covered inside `Requirement_Gathering.md` (the 3-bucket model); not a clean gap.
- Consistency vs availability trade-off framing → `CAP_Theorem_PACELC.md` (here) plus depth in `05_Databases_Storage/CAP_Theorem.md`, `PACELC_Theorem.md`, `Consistency_Models.md`
- Back-pressure & throttling → `09_Message_Queues_Event_Streaming/Backpressure_Throttling.md`
- Horizontal vs vertical scaling mechanics, autoscaling → `07_Load_Balancing_Scalability/Auto_scaling_Horizontal_vs_Vertical.md`
- SPOF analysis, redundancy patterns → `03_High_Level_Design/High_Availability_Design.md`, `07_Load_Balancing_Scalability/Fault_Tolerance_vs_Resilience.md`
- Performance metrics (percentiles, p99) → `06_Caching_Performance/Performance_Metrics.md`; latency vs throughput vs bandwidth → `14_Performance_Optimization_Scalability/Latency_Throughput_Bandwidth.md`
- Disaster-recovery targets (RPO/RTO) → `13_Cloud_Native_Infrastructure/Disaster_Recovery_RPO_RTO.md`
- SLO/SLI operationalization (error-budget policy, burn rates) → `12_Monitoring_Logging_Observability/SLO_SLI_Error_Budgets.md`

## Folder housekeeping

- **CAP/PACELC triple coverage** — `CAP_Theorem_PACELC.md` (here) overlaps with `05_Databases_Storage/CAP_Theorem.md` *and* `05_Databases_Storage/PACELC_Theorem.md`: three files across two folders for the same two theorems. Keep this one as the interview trade-off-framing version and make the 05 pair the database-depth versions, with explicit cross-links stating the split — or consolidate the 05 pair into one file.
- README.md exists and indexes all 12 topics; no other structural issues.
