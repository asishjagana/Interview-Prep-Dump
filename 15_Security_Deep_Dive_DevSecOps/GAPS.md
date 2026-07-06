# Gap Analysis — 15_Security_Deep_Dive_DevSecOps

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 7 (5 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Zero_Trust_Architecture.md | Identity as the perimeter, micro-segmentation |
| IAM_Policies_Cloud_Security.md | Least privilege, policy anatomy, blast radius |
| API_Security_Scopes_mTLS.md | OAuth2 scopes, mTLS for service-to-service traffic |
| Secrets_Management_Vault_KMS.md | Dynamic secrets, rotation, centralized issuance |
| KMS_HSM_Key_Management.md | Envelope encryption, root-key custody |
| Intrusion_Detection_IDS_IPS.md | Signature vs anomaly detection, detect vs block |
| Pentesting_vs_Vuln_Scanning.md | Offensive validation depth vs breadth |
| OWASP_Top_10_Deep_Dive.md | The canonical app-vulnerability list |
| Secure_Software_Development_Lifecycle_SSDLC.md | Shift-left: SAST/DAST/dep-scan in the pipeline |
| Compliance_Frameworks_SOC2_GDPR.md | SOC2, GDPR, HIPAA — proving security |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **WAF_DDoS_Mitigation_Architecture.md** — WAF placement and rule models, volumetric (L3/4) vs application-layer (L7) DDoS, scrubbing centers, anycast absorption, Cloudflare/AWS Shield patterns. (Promised by `11_Security_Identity/GAPS.md`, which keeps request-level rate limiting and defers the infrastructure-scale defense here; WAF today is a single passing mention in SSDLC's deploy stage.)
2. **Threat_Modeling_STRIDE.md** — STRIDE/DREAD, attack trees, data-flow diagrams, trust boundaries, when to threat-model in the lifecycle. (SSDLC names threat modeling as its design-stage activity but never teaches the method — a standard Staff+ interview probe.)
3. **Supply_Chain_Security_SBOM.md** — Dependency scanning, SBOM formats (SPDX/CycloneDX), SLSA levels, artifact signing (Sigstore/cosign), typosquatting and build-pipeline compromise (SolarWinds/xz lessons). (Dep-scanning gets one line in SSDLC; the supply-chain attack class has no home despite being a top post-2020 interview topic.)
4. **Container_Kubernetes_Security.md** — Image scanning and minimal base images, pod security standards, K8s RBAC, network policies, runtime detection (Falco), admission controllers. (Zero coverage in 13 or 15; `13_Cloud_Native_Infrastructure/GAPS.md` proposed an optional registry/image file and flagged the overlap — this folder takes ownership of the security treatment, 13 keeps registry mechanics.)
5. **Security_Incident_Response.md** — IR lifecycle (detect → contain → eradicate → recover → learn), SIEM/SOC operations, playbooks/runbooks, forensics basics, breach notification duties under GDPR/SOC2. (SIEM appears only as the destination in IDS/IPS; nothing covers what happens after detection.)

### Optional / nice-to-have

6. **Data_Privacy_Engineering_PII.md** — Data classification, tokenization vs anonymization vs pseudonymization, right-to-be-forgotten deletion pipelines, data retention. (Compliance_Frameworks covers the legal *what*; this is the engineering *how*.)
7. **Network_Segmentation_Microsegmentation.md** — Firewalls/security zones, east-west vs north-south traffic, microsegmentation in practice. (Partially inside Zero_Trust_Architecture.md; split out only if that file stays philosophy-level. Cloud-side security groups/NACLs belong to 13's proposed `Cloud_Networking_VPC_Subnets.md`.)

## Deliberately NOT flagged (covered in other folders)

- Authentication/authorization fundamentals, OAuth2/OIDC, JWT, RBAC/ABAC → `11_Security_Identity`
- PKI/TLS certificate machinery → proposed in 11's gap list (`PKI_TLS_Certificates.md`); this folder's mTLS file consumes it
- Session, MFA/WebAuthn, CORS/browser security → proposed in 11's gap list
- Request-level rate limiting & DoS basics → `11_Security_Identity/Rate_Limiting_DoS_Protection.md` and `02_API_Interface_Design/Rate_Limiting.md`
- Audit logging & security-event pipelines → proposed in `12_Monitoring_Logging_Observability/GAPS.md` (`Audit_Logging_Security_Events.md`); cross-link from the incident-response file once both exist
- Config distribution with Vault → `10_Microservices_Distributed_Systems/Configuration_Management_Vault.md` (10 = config plumbing, 15 = secrets lifecycle)
- CI/CD pipeline mechanics that SSDLC hooks into → `13_Cloud_Native_Infrastructure/CI_CD_Pipelines.md`

## Folder housekeeping

- **Secrets_Management_Vault_KMS.md vs KMS_HSM_Key_Management.md vs 10's Configuration_Management_Vault.md** — three files share the Vault/KMS surface. Delineation is workable (10 = config distribution, 15a = secrets lifecycle, 15b = key custody/envelope encryption) but should be stated explicitly in each file's intro with cross-links.
- **Coordinate with `13_Cloud_Native_Infrastructure/GAPS.md`** — its optional `Container_Registry_Image_Security.md` overlaps core gap #4 above; build one file (here) and cross-reference from 13 rather than two.
- When gap files are added, update README's mermaid map, file index, and study paths (they currently hard-code "the 10 topics").
