# Gap Analysis — 16_Emerging_Trends_Miscellaneous

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 8 (5 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| AI_for_System_Design.md | AIOps, predictive scaling, autonomous tuning |
| LLM_Ops_Patterns.md | RAG pipeline, prompt chaining, guardrails, LLM caching |
| Edge_AI_Federated_Learning.md | On-device inference, federated training |
| Local_First_Architecture.md | Device-as-source-of-truth, CRDT merge, sync relays |
| Web3_Blockchain_Architecture.md | Distributed ledgers, consensus, smart contracts |
| WebAssembly_Wasm_Backend.md | Sandboxed near-native compute, WASI, ms cold starts |
| LowCode_NoCode_Architecture.md | Metadata-driven app platforms, governance |
| Multi_Cloud_Inter_Cloud_Networking.md | Interconnects, SD-WAN, egress economics |
| Quantum_Computing_Readiness.md | Post-quantum crypto migration, harvest-now-decrypt-later |
| Sustainability_Green_Computing.md | Carbon-aware scheduling, efficiency as ESG lever |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **AI_Agent_Architectures.md** — Tool use/function calling, agent loops (plan → act → observe), multi-agent orchestration, MCP-style tool protocols, agent memory, evaluation and guardrails for autonomous systems. (LLM_Ops_Patterns.md stops at RAG/prompt-chaining pipelines; agentic systems are the highest-velocity interview topic of 2025–26 and have zero coverage.)
2. **RAG_Architecture_Deep_Dive.md** — Chunking strategies, embedding models, hybrid (keyword + vector) search, re-ranking, RAG evaluation (groundedness/recall), when RAG loses to fine-tuning or long context. (LLM_Ops_Patterns.md introduces RAG as one stage of its pipeline; the design-a-RAG-system interview needs its own treatment. The datastore itself is `05_Databases_Storage`'s proposed `Vector_Databases.md` — cross-link, don't duplicate.)
3. **Durable_Execution_Workflow_Engines.md** — Temporal/Cadence, AWS Step Functions, event-sourced workflow state, deterministic replay, human-in-the-loop steps. (Zero repo coverage; `10_Microservices_Distributed_Systems/Orchestration_vs_Choreography.md` names the orchestrator pattern but no engine-level file exists, and 10's GAPS didn't claim it.)
4. **Real_Time_Collaboration_OT_CRDT.md** — Operational Transform vs CRDTs, presence/awareness, server reconciliation, offline + concurrent editing at Google-Docs/Figma scale. ("Design a collaborative editor" is a classic interview; Local_First_Architecture.md covers CRDT merge for single-user sync but not the multi-user collaboration system.)
5. **Platform_Engineering_Internal_Developer_Platforms.md** — Golden paths, Backstage-style service catalogs, self-service infra, platform-as-product thinking, cognitive-load reduction. (Zero mentions repo-wide; the successor conversation to "DevOps" in senior interviews.)

### Optional / nice-to-have

6. **IoT_Digital_Twins_Architecture.md** — Device fleets, telemetry ingestion at scale, digital-twin state models, OTA updates. (MQTT protocol itself is proposed in `08_Communication_Protocols/GAPS.md`; this would be the system-level view.)
7. **eBPF_Kernel_Programmability.md** — eBPF for networking (Cilium), observability, and security (Falco) without kernel modules; why it's displacing sidecars. (Ties into 13's service-mesh housekeeping note on ambient/sidecar-less mesh.)
8. **AR_VR_Spatial_Computing_Infrastructure.md** — Latency budgets for immersion, edge rendering/split compute, spatial data sync.

## Deliberately NOT flagged (covered in other folders)

- Vector databases as a datastore → proposed in `05_Databases_Storage/GAPS.md` (`Vector_Databases.md`); gap #2 above owns the pipeline, 05 owns the store
- Edge computing runtime (CDN functions, V8 isolates) → `13_Cloud_Native_Infrastructure/Edge_Computing_Cloudfront_Functions.md`
- Serverless/FaaS fundamentals → `13_Cloud_Native_Infrastructure/Serverless_Computing_Lambda.md`
- MQTT/AMQP and IoT wire protocols → proposed in `08_Communication_Protocols/GAPS.md` (`MQTT_AMQP_IoT_Protocols.md`)
- WebRTC (the transport under many collaboration/metaverse designs) → proposed in `08_Communication_Protocols/GAPS.md` (`WebRTC.md`)
- Chaos engineering (sometimes bucketed as "emerging practice") → proposed in `13_Cloud_Native_Infrastructure/GAPS.md` (`Chaos_Engineering.md`)
- Post-quantum crypto mechanics → stays here in Quantum_Computing_Readiness.md; key-custody plumbing is `15_Security_Deep_Dive_DevSecOps/KMS_HSM_Key_Management.md`

## Folder housekeeping

- **Multi_Cloud_Inter_Cloud_Networking.md vs `13_Cloud_Native_Infrastructure/Hybrid_Multi_Cloud_Strategies.md`** — 13 owns the strategy (lock-in, DR, workload placement), 16 owns the networking (interconnects, SD-WAN, egress). The split is real but neither file says so; add explicit scope notes and cross-links.
- If gap #1/#2 are added, consider renaming LLM_Ops_Patterns.md's intro to position it as the pipeline/ops layer between them (RAG deep-dive ← LLM Ops → agents) so the three files read as one arc.
- When gap files are added, update README's mermaid map, file index, and study paths (they currently hard-code "the 10 topics").
