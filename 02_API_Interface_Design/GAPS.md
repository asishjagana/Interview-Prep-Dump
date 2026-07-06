# Gap Analysis — 02_API_Interface_Design

> Audit date: 2026-07-05. Existing files: 11. Missing (proposed): 7 (4 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| API_Design_Fundamentals.md | The API-as-contract mindset, the 4 pillars |
| REST_API_Design.md | Resources × verbs, status codes, pragmatic REST |
| GraphQL.md | Client-specified shape, N+1/DataLoader |
| gRPC.md | Typed RPC, Protobuf, HTTP/2, internal APIs |
| API_Versioning.md | Evolving without breaking, additive changes |
| Pagination_Filtering.md | Cursor vs offset, bounding every list |
| Idempotency.md | Safe retries, Idempotency-Key pattern |
| Authentication.md | Who are you — hashing, sessions, JWTs |
| Authorization.md | What can you do — object-level checks, IDOR, tenant scoping |
| Rate_Limiting.md | Token bucket, 429 + Retry-After, shared Redis |
| API_Gateway.md | The front door, cross-cutting concerns at the edge |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **API_Documentation_OpenAPI.md** — OpenAPI/Swagger as the machine-readable contract, spec-first vs code-first, docs-as-product (reference + guides + changelogs), consumer-driven contract testing in CI, AsyncAPI for event contracts. (Only passing mentions today in `API_Design_Fundamentals.md` and `gRPC.md`; "the contract is the product" deserves the file that shows *how* the contract is written and enforced.)
2. **API_Error_Handling_Design.md** — Designing the error surface: one error envelope, stable machine-readable codes, RFC 7807/9457 `application/problem+json`, retryable vs non-retryable classification, partial-failure reporting, error-message security (don't leak internals). (Partial coverage exists — the envelope in `API_Design_Fundamentals.md` and status-code honesty in `REST_API_Design.md` — but the error taxonomy treatment is scattered; consolidate and cross-link.)
3. **Async_APIs_Long_Running_Operations.md** — When a request outlives the connection: `202 Accepted` + operation/status resource, polling vs callback (webhook) completion, progress reporting, cancellation, TTL of results, AsyncAPI/job-queue backing. Classic interview follow-up ("the report takes 10 minutes to generate — now what?") with no dedicated file anywhere.
4. **Bulk_Batch_APIs.md** — Batch endpoints and bulk import/export: all-or-nothing vs per-item results (`207 Multi-Status`), partial-failure semantics, idempotency of batches, payload limits and chunking, rate-limit interaction, bulk async jobs. (Only a one-word mention in `API_Design_Fundamentals.md` today.)

### Optional / nice-to-have

5. **HATEOAS_Hypermedia.md** — Hypermedia controls, the Richardson Maturity Model, why pragmatic REST skips level 3, when links genuinely pay off (workflow/state-machine APIs). (Currently a Q&A answer inside `REST_API_Design.md`; split out only if you want the maturity-model depth.)
6. **CORS_Same_Origin_Policy.md** — Same-origin policy, preflight requests, credentialed requests, common misconfigurations (`*` + credentials), CORS ≠ security. (One-line mention in `15_Security_Deep_Dive_DevSecOps/API_Security_Scopes_mTLS.md`; straddles this folder and 11 — place here since it shapes browser-facing API design.)
7. **SDK_Client_Design.md** — Designing the client side of the contract: generated vs handwritten SDKs, baked-in retries/backoff/idempotency keys, pagination iterators, versioning SDKs alongside the API.

## Deliberately NOT flagged (covered in other folders)

- Webhooks (callbacks-as-API pattern) → `08_Communication_Protocols/Webhooks.md`
- REST vs GraphQL vs gRPC comparison → `08_Communication_Protocols/REST_vs_GraphQL_vs_gRPC.md` (styles individually covered here)
- WebSockets, SSE, long/short polling (realtime API transports) → `08_Communication_Protocols`
- OAuth2/OIDC, JWT internals, SSO/SAML → `11_Security_Identity`
- API security depth (scopes, mTLS) → `15_Security_Deep_Dive_DevSecOps/API_Security_Scopes_mTLS.md`
- Rate limiting as DoS protection → `11_Security_Identity/Rate_Limiting_DoS_Protection.md`
- Binary serialization formats (Protobuf/Avro) → `14_Performance_Optimization_Scalability/Binary_Protocols_Protobuf_Avro.md`
- HTTP caching semantics (ETag, Cache-Control) → `06_Caching_Performance/HTTP_Caching.md`
- Backend-for-Frontend (BFF) → proposed in `10_Microservices_Distributed_Systems/GAPS.md`

## Folder housekeeping

- **Duplicate filename across folders** — `API_Versioning.md` exists both here and in `08_Communication_Protocols/`. Two files, same topic name, no cross-links. Keep this one (versioning is a contract concern, not a wire-protocol concern), demote the 08 copy to a redirect stub or merge it in.
- **Gateway topic spread across three folders** — `API_Gateway.md` (here), `07_Load_Balancing_Scalability/LB_vs_Reverse_Proxy_vs_API_Gateway.md`, and `10_Microservices_Distributed_Systems/API_Gateway_Sidecar_Pattern.md` each re-explain the gateway. The angles differ (edge design / traffic comparison / microservice pattern) — acceptable, but add explicit "which file for what" cross-links to prevent drift.
