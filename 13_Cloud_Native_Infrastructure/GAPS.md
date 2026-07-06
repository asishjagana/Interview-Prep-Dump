# Gap Analysis — 13_Cloud_Native_Infrastructure

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 8 (5 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Containers_vs_VMs_Docker.md | Namespaces/cgroups vs hypervisors, container fundamentals |
| Kubernetes_Objects_Pods_Services.md | Pod/Deployment/Service model, declarative self-healing |
| Service_Mesh_Deep_Dive_Istio.md | Sidecar data plane, control plane, mTLS (see housekeeping) |
| Infrastructure_as_Code_Terraform.md | Declarative, idempotent infra; state management |
| CI_CD_Pipelines.md | Build/test/ship automation, canary rollout, auto-rollback |
| Immutable_Infrastructure_Pattern.md | Golden images, kill-and-replace, config-drift elimination |
| Serverless_Computing_Lambda.md | FaaS, scale-to-zero, cold starts |
| Edge_Computing_Cloudfront_Functions.md | PoP execution, V8 isolates, latency floor |
| Disaster_Recovery_RPO_RTO.md | RPO/RTO, backup → pilot light → warm standby → active/active |
| Hybrid_Multi_Cloud_Strategies.md | Lock-in avoidance, hybrid connectivity, cloud-agnostic glue |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Deployment_Strategies_Blue_Green_Canary.md** — Blue-green, canary, rolling, and shadow deployments; feature flags and dark launches; database migrations under zero-downtime deploys. (Touched inside CI_CD_Pipelines.md, but folder 10's audit explicitly deferred the dedicated treatment to this folder; it is the single most-asked "how do you ship safely?" topic.)
2. **Kubernetes_Scheduling_Autoscaling_HPA_VPA.md** — Requests/limits and QoS classes, bin-packing and the scheduler, taints/tolerations/affinity, HPA vs VPA vs Cluster Autoscaler (and KEDA for event-driven scale). (`07_Load_Balancing_Scalability/Auto_scaling_Horizontal_vs_Vertical.md` covers VM-level autoscaling; the Kubernetes-native layer has no file.)
3. **GitOps_Helm_ArgoCD.md** — Helm charts/templating, GitOps pull-based reconciliation, ArgoCD/Flux, drift detection, environment promotion via git. (Zero mentions of Helm/GitOps/ArgoCD in any numbered folder — a notable hole for a "cloud-native" section.)
4. **Cloud_Networking_VPC_Subnets.md** — VPCs, public/private subnets, security groups vs NACLs, NAT/internet gateways, VPC peering and PrivateLink. (Essentially absent from the repo; `16_Emerging_Trends_Miscellaneous/Multi_Cloud_Inter_Cloud_Networking.md` covers only the inter-cloud angle.)
5. **Chaos_Engineering.md** — Chaos Monkey/Litmus, fault injection, steady-state hypotheses, game days, blast-radius control. (Scattered "fault injection" mentions only; interviewers use it to probe resilience maturity.)

### Optional / nice-to-have

6. **Managed_Services_vs_Self_Hosted.md** — Build-vs-buy for databases/queues/K8s (RDS vs self-run Postgres, MSK vs self-run Kafka, EKS vs kubeadm); TCO, undifferentiated heavy lifting, lock-in vs velocity.
7. **Container_Registry_Image_Security.md** — Registries, image scanning (Trivy), image signing/provenance (cosign, SLSA), minimal base images. (Could alternatively live in `15_Security_Deep_Dive_DevSecOps` — nothing there covers it today; coordinate to avoid a double file.)
8. **Kubernetes_Operators_CRDs.md** — CRDs, the operator/reconciliation pattern, when to build an operator vs a Helm chart.

## Deliberately NOT flagged (covered in other folders)

- Cost optimization / FinOps → `01_Planning_Foundations/Cost_Estimation_FinOps.md`
- Auto-scaling fundamentals (horizontal vs vertical) → `07_Load_Balancing_Scalability/Auto_scaling_Horizontal_vs_Vertical.md`
- Secrets & config distribution → `10_Microservices_Distributed_Systems/Configuration_Management_Vault.md` and `15_Security_Deep_Dive_DevSecOps/Secrets_Management_Vault_KMS.md`
- IAM and cloud account security → `15_Security_Deep_Dive_DevSecOps/IAM_Policies_Cloud_Security.md`
- Zero-trust networking → `15_Security_Deep_Dive_DevSecOps/Zero_Trust_Architecture.md`
- CDN / edge caching (vs edge compute here) → `06_Caching_Performance/CDN.md`
- Inter-cloud networking → `16_Emerging_Trends_Miscellaneous/Multi_Cloud_Inter_Cloud_Networking.md`

## Folder housekeeping

- **Service_Mesh_Deep_Dive_Istio.md is largely redundant with `10_Microservices_Distributed_Systems/Service_Mesh_Istio_Linkerd.md`** — both files have the identical 16-section structure and cover the same ground (Envoy sidecar data plane, control plane, mTLS, canary weights) at near-identical length (111 vs 117 lines); the "Deep Dive" adds no depth beyond the folder-10 file. Either differentiate this one into a true deep dive (VirtualService/DestinationRule traffic model, ambient mesh/sidecar-less mode, mesh performance overhead) or merge into folder 10 and drop it here.
- When gap files are added, update README's mermaid map, file index, and study paths (they currently hard-code "the 10 topics").
