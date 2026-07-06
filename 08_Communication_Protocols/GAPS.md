# Gap Analysis — 08_Communication_Protocols

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 8 (5 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| TCP_vs_UDP.md | Transport guarantees vs speed |
| HTTP_Versions.md | HTTP/1.1 → 2 → 3/QUIC, multiplexing, HOL blocking |
| REST_vs_GraphQL_vs_gRPC.md | API style comparison |
| SOAP_vs_REST.md | WSDL contracts vs resource style |
| API_Versioning.md | URI / header / media-type versioning |
| WebSockets.md | Full-duplex persistent connections |
| Server_Sent_Events.md | One-way server→client streaming |
| Long_Polling_vs_Short_Polling.md | Pre-WebSocket fallbacks |
| Webhooks.md | Provider-initiated callbacks, retries |
| Message_Queues_PubSub.md | Broker decoupling, delivery guarantees |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **DNS_Deep_Dive.md** — Resolution path (stub → recursive → root/TLD/authoritative), record types (A/AAAA/CNAME/NS/TXT/SRV), TTL and caching layers, recursive vs iterative queries, DNS as the first hop of every design. (`06_Caching_Performance/GSLB.md` uses DNS purely as a routing lever; how DNS itself works has no dedicated file anywhere.)
2. **TLS_SSL_Handshake.md** — TLS 1.2 vs 1.3 handshakes, RTT cost, session resumption/0-RTT, certificate chains and CAs, SNI/ALPN, where TLS terminates. (Partial coverage in `11_Security_Identity/Encryption_Rest_vs_Transit.md` from a data-protection angle; the wire-protocol treatment — the one that explains connection-setup latency and pairs with `HTTP_Versions.md` — belongs here. Cross-link the two.)
3. **WebRTC.md** — Peer-to-peer media, signaling, STUN/TURN/ICE NAT traversal, SFU vs MCU topologies. (Zero mentions in the entire repo; the "design a video call / Zoom" staple, and the missing fifth member of the real-time push family in ⑥–⑨.)
4. **Video_Streaming_Protocols_HLS_DASH.md** — HTTP adaptive bitrate streaming, segmenting and manifests, live vs VOD, CDN delivery, latency tiers vs WebRTC. (Zero coverage; required for "design YouTube/Netflix/Twitch".)
5. **MQTT_AMQP_IoT_Protocols.md** — MQTT pub/sub with QoS 0/1/2, retained messages and LWT, AMQP vs MQTT, CoAP; when device constraints rule out HTTP. (MQTT gets one passing mention in `Message_Queues_PubSub.md`; the IoT-design question has no protocol answer in the repo.)

### Optional / nice-to-have

6. **OSI_Model_TCP_IP_Stack.md** — The layer model as shared vocabulary: what actually happens at L2–L7 when a request is sent. (Referenced by name in `07_Load_Balancing_Scalability/Layer_4_vs_Layer_7_Load_Balancing.md` but never explained anywhere.)
7. **RPC_Fundamentals_Thrift.md** — RPC as a paradigm: stubs, IDLs, marshalling, Thrift vs gRPC vs JSON-RPC lineage. (gRPC itself is covered in `02_API_Interface_Design/gRPC.md`; the generic-RPC framing is the gap.)
8. **HTTP_Semantics_Headers_Deep_Dive.md** — Content negotiation, cookies, CORS preflight, conditional requests, keep-alive — the header-level mechanics between versions (HTTP_Versions.md) and API style (REST files). (Methods/status codes are already well covered in `02_API_Interface_Design/REST_API_Design.md`, hence optional.)

## Deliberately NOT flagged (covered in other folders)

- QUIC / HTTP/3 → already covered in-folder by `HTTP_Versions.md` (verified: 3/QUIC treated as a first-class section)
- gRPC deep-dive (protobuf, streaming modes) → `02_API_Interface_Design/gRPC.md`
- REST design, methods, status codes → `02_API_Interface_Design/REST_API_Design.md`
- GraphQL deep-dive → `02_API_Interface_Design/GraphQL.md`
- Idempotency of retried calls/webhooks → `02_API_Interface_Design/Idempotency.md`
- mTLS, API scopes → `15_Security_Deep_Dive_DevSecOps/API_Security_Scopes_mTLS.md`
- Encryption in transit (the security angle of TLS) → `11_Security_Identity/Encryption_Rest_vs_Transit.md`
- Broker internals: Kafka, ordering, consumer groups, exactly-once → `09_Message_Queues_Event_Streaming`
- Binary serialization formats (Protobuf/Avro on the wire) → `06_Caching_Performance/Data_Compression_Serialization.md` and `14_Performance_Optimization_Scalability/Binary_Protocols_Protobuf_Avro.md`

## Folder housekeeping

- **Duplicate file across folders:** `API_Versioning.md` exists BOTH here and in `02_API_Interface_Design/`. Versioning is a contract-design concern, not a wire-protocol one — 02 is the natural owner; keep one canonical file and leave a cross-link (or delete this copy) after merging any unique content.
- `Message_Queues_PubSub.md` overlaps the whole of `09_Message_Queues_Event_Streaming`. Fine as an intro/on-ramp, but it should state that scope explicitly and link forward to 09 so the two don't drift.
- Minor: `SOAP_vs_REST.md` and `REST_vs_GraphQL_vs_gRPC.md` both re-cover REST as a comparison baseline; acceptable, but keep the REST fundamentals in 02 and comparisons only here.
