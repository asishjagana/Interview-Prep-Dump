# Gap Analysis — 07_Load_Balancing_Scalability

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 4 (2 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Load_Balancing_Algorithms.md | Round robin, weighted, least-connections, least-response-time |
| Layer_4_vs_Layer_7_Load_Balancing.md | Transport vs application-layer routing |
| Hardware_vs_Software_Load_Balancers.md | ASIC appliances vs Nginx/HAProxy/Envoy/cloud ALB |
| LB_vs_Reverse_Proxy_vs_API_Gateway.md | Edge component responsibilities |
| Consistent_Hashing.md | Hash ring, virtual nodes, ~1/N remapping |
| Health_Checks_Failover.md | Active vs passive checks, failover thresholds |
| Fault_Tolerance_vs_Resilience.md | Masking vs recovering from failure |
| High_Availability_Active_Active_vs_Active_Passive.md | HA topologies and failover trade-offs |
| Auto_scaling_Horizontal_vs_Vertical.md | Scaling axes, reactive vs predictive policies |
| Sticky_Sessions_vs_Stateless_Design.md | Session affinity vs externalized state (Redis/JWT) |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Load_Shedding_Graceful_Degradation.md** — Admission control at the edge, prioritized request dropping, brownout/feature degradation, queue length limits, overload vs autoscaling lag. (Only scattered one-line mentions across 01/02/03/09/10; the "what happens when the fleet is at 100% and autoscaling is 90 seconds away" question has no home. Distinct from `09_Message_Queues_Event_Streaming/Backpressure_Throttling.md`, which is broker/consumer-side — cross-link the two.)
2. **Load_Balancer_High_Availability.md** — "Who balances the balancer": DNS round robin at the top, anycast, VRRP/keepalived floating IPs, ECMP/BGP, LB pairs and connection-state sync, connection draining during failover. (The classic "isn't your LB a single point of failure?" follow-up; VRRP/keepalived appear nowhere in the repo.)

### Optional / nice-to-have

3. **Client_Side_vs_Server_Side_Load_Balancing.md** — Client-side LB (gRPC pick-first/round-robin, lookaside LB, Ribbon-style libraries) vs a dedicated middle proxy; where the service-mesh sidecar sits on that spectrum. (Sidecar traffic management is touched in 10/13 mesh files, but the client-vs-server LB framing itself is undocumented.)
4. **Scale_Cube_XYZ_Axis_Scaling.md** — AKF scale cube: X (cloning), Y (functional decomposition), Z (data partitioning) as the unifying vocabulary tying this folder to 03 (decomposition) and 05 (sharding).

## Deliberately NOT flagged (covered in other folders)

- DNS load balancing, GeoDNS, anycast global routing → `06_Caching_Performance/GSLB.md` (cross-link it from this folder's README)
- Rate limiting at the LB/gateway → `02_API_Interface_Design/Rate_Limiting.md` and `11_Security_Identity/Rate_Limiting_DoS_Protection.md`
- Read-replica routing / read-write splitting → `05_Databases_Storage/Replication.md` and `05_Databases_Storage/SQL_vs_NoSQL_Scaling.md`
- Blue-green / canary traffic shifting → deployment-strategy topic; belongs in `13_Cloud_Native_Infrastructure` (already deferred there by 10's audit — no dedicated file yet, but it is 13's gap, not this folder's)
- Circuit breakers, bulkheads, retries → `10_Microservices_Distributed_Systems/Circuit_Breaker_Bulkhead.md`
- Backpressure & throttling in async pipelines → `09_Message_Queues_Event_Streaming/Backpressure_Throttling.md`
- CDN offload of edge traffic → `06_Caching_Performance/CDN.md`
- Kubernetes autoscaling primitives (HPA, ASGs as managed services) → `13_Cloud_Native_Infrastructure/Kubernetes_Objects_Pods_Services.md`
- Disaster recovery, RPO/RTO → `13_Cloud_Native_Infrastructure/Disaster_Recovery_RPO_RTO.md`

## Folder housekeeping

- No duplicate files. Folder is internally coherent; the ten files map cleanly onto mechanics → topology → resilience → scaling.
- `High_Availability_Active_Active_vs_Active_Passive.md` overlaps `03_High_Level_Design/High_Availability_Design.md` in theme — acceptable (03 is architectural, this is topology-level), but the two should cross-link explicitly.
- The folder's global-routing story (multi-region, GeoDNS) lives in `06_Caching_Performance/GSLB.md`; consider a pointer in this README's file index so readers looking for "global load balancing" here don't conclude it's missing.
