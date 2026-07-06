# Gap Analysis — 12_Monitoring_Logging_Observability

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 6 (3 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Monitoring_Logging_Tracing_Three_Pillars.md | Metrics/logs/traces overview |
| Golden_Signals_Latency_Traffic_Errors.md | Latency, traffic, errors, saturation |
| Metrics_Collection_Prometheus_Grafana.md | Pull-based metrics & visualization |
| Log_Management_ELK_Stack.md | Log ingest, indexing, search (ELK) |
| Distributed_Tracing_OpenTelemetry.md | OTel instrumentation, context propagation, collector |
| Profiling_Distributed_Profiling.md | Code-level & continuous profiling |
| SLO_SLI_Error_Budgets.md | Reliability targets & error budgets |
| Synthetic_Monitoring_Heartbeats.md | Scripted probes & dead-man's switches |
| Centralized_Dashboard_Design.md | Hierarchical dashboard design |
| Alerting_Strategies_OnCall_Management.md | Symptom-based alerting & on-call |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Structured_Logging_Sampling_Strategies.md** — JSON/structured logs, correlation/request IDs, log levels done right, and sampling at scale (log sampling, head-based vs tail-based trace sampling, cost control at millions of QPS). (Log_Management_ELK_Stack.md is pipeline/tooling-focused and Distributed_Tracing_OpenTelemetry.md doesn't cover sampling — the "how do you log at 1M QPS without bankrupting yourself" question has no home.)
2. **Metrics_Design_Cardinality.md** — Counter/gauge/histogram/summary semantics, RED vs USE methods, label design, and high-cardinality explosions that kill a TSDB. (Cardinality gets only two passing mentions in Metrics_Collection_Prometheus_Grafana.md; metric-design judgment is a standard senior-level probe.)
3. **Audit_Logging_Security_Events.md** — Immutable audit trails, who-did-what-when logs, tamper evidence, retention, and shipping security events to a SIEM. (Compliance frameworks in `15_Security_Deep_Dive_DevSecOps/Compliance_Frameworks_SOC2_GDPR.md` *require* audit logs but nothing teaches how to build them; cross-link 15.)

### Optional / nice-to-have

4. **Anomaly_Detection_Alert_Correlation.md** — Static thresholds vs baselines vs ML-based detection, alert dedup/correlation, noise reduction. (AIOps angle partially covered by `16_Emerging_Trends_Miscellaneous/AI_for_System_Design.md`; security-side anomaly detection by 15's IDS/IPS file — this would be the observability-native treatment.)
5. **Real_User_Monitoring_RUM.md** — Field data vs synthetic probes, Core Web Vitals, frontend error tracking; the natural complement to Synthetic_Monitoring_Heartbeats.md.
6. **APM_Tools_Landscape.md** — Datadog/New Relic/Dynatrace vs the OSS stack (Prometheus+Grafana+Loki+Tempo), build-vs-buy trade-offs, cost models.

## Deliberately NOT flagged (covered in other folders)

- Cross-service tracing with Jaeger → `10_Microservices_Distributed_Systems/Distributed_Tracing_Jaeger.md` (this folder's OTel file is the standard/instrumentation side; cross-link, don't duplicate)
- SLA/SLO/SLI definitions → `01_Planning_Foundations/SLA_SLO_SLI.md` (in-folder SLO_SLI_Error_Budgets.md operationalizes them)
- Capacity monitoring & forecasting → `01_Planning_Foundations/Capacity_Planning.md`
- Performance testing & benchmarking → `14_Performance_Optimization_Scalability/Performance_Testing_Benchmarking.md` (plus `06_Caching_Performance/Profiling_Benchmarking.md`)
- Health checks & failover probes → `07_Load_Balancing_Scalability/Health_Checks_Failover.md`
- AIOps / AI-driven operations → `16_Emerging_Trends_Miscellaneous/AI_for_System_Design.md`
- Chaos engineering & game days → proposed as `13_Cloud_Native_Infrastructure`'s gap `Chaos_Engineering.md` (13 owns the practice alongside deployment strategies; this folder owns the observability that validates it — cross-link once written)

## Folder housekeeping

- **Distributed_Tracing_OpenTelemetry.md overlaps `10_Microservices_Distributed_Systems/Distributed_Tracing_Jaeger.md`** — both walk trace ID + spans across services with Jaeger as backend. Delineate explicitly (10 = the distributed-systems problem + Jaeger; 12 = the OTel standard, instrumentation, collector pipeline) and cross-link the two.
- **Files are still on the legacy 16-section format** — the README promises the root TEMPLATE.md structure with the 🏋️/🎤/🎯 mastery sections, but the topic files don't have them yet; migrate as done for folders 01–05.
