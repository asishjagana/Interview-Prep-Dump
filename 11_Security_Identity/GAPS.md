# Gap Analysis — 11_Security_Identity

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 6 (4 core + 2 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Authentication_vs_Authorization.md | AuthN vs AuthZ fundamentals & placement |
| JWT_Basics_Best_Practices.md | Token anatomy, signing, verification, rotation |
| OAuth2_OpenID_Connect_OIDC.md | Delegated access & identity layer |
| SSO_and_SAML.md | Enterprise identity federation |
| RBAC_vs_ABAC.md | Authorization models |
| Hashing_and_Salting.md | Password storage (Argon2/BCrypt, salts) |
| Encryption_Rest_vs_Transit.md | TLS in transit, AES/TDE at rest |
| SQL_Injection_Input_Validation.md | Injection & input validation |
| XSS_and_CSRF.md | Browser-side exploits & defenses |
| Rate_Limiting_DoS_Protection.md | Abuse throttling & DoS basics |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Session_Management_Cookie_Security.md** — Server-side sessions vs stateless tokens, session fixation/hijacking, cookie attributes (HttpOnly, Secure, SameSite), session invalidation & logout-everywhere. (Session-vs-token comparison sits briefly in `02_API_Interface_Design/Authentication.md` §3 and cookie flags appear only as CSRF countermeasures in XSS_and_CSRF.md — no dedicated treatment.)
2. **MFA_Passwordless_WebAuthn.md** — TOTP/SMS/push factors and their trade-offs, MFA fatigue attacks, passkeys/WebAuthn/FIDO2, passwordless flows. (Only passing MFA mentions in 02's Authentication.md and SSO_and_SAML.md; a staple "how do you harden login?" follow-up.)
3. **PKI_TLS_Certificates.md** — TLS handshake, certificate chains & CAs, X.509, cert issuance/rotation (ACME/Let's Encrypt), pinning, expiry outages. (Encryption_Rest_vs_Transit.md *uses* TLS but doesn't teach the PKI machinery; mTLS usage lives in 15 — this file is the fundamentals both lean on.)
4. **CORS_Browser_Security_Model.md** — Same-origin policy, CORS preflight & headers, Content-Security-Policy, security headers (HSTS, X-Frame-Options). (CSP appears only as an XSS second layer in XSS_and_CSRF.md; CORS misconfiguration is a very common interview and real-world topic with no home today.)

### Optional / nice-to-have

5. **API_Keys_Service_to_Service_Auth.md** — API keys vs OAuth2 client-credentials vs signed requests (HMAC/SigV4-style), key issuance/rotation/revocation for machine-to-machine callers. (OAuth2 scopes + mTLS for service traffic are in `15_Security_Deep_Dive_DevSecOps/API_Security_Scopes_mTLS.md`; this covers the simpler key-based patterns interviewers often probe first.)
6. **Bot_Detection_CAPTCHA_Challenges.md** — CAPTCHA/proof-of-work challenges, device fingerprinting, credential-stuffing defense; complements Rate_Limiting_DoS_Protection.md on the abuse-prevention side.

## Deliberately NOT flagged (covered in other folders)

- OAuth2 scopes & mTLS for service-to-service traffic → `15_Security_Deep_Dive_DevSecOps/API_Security_Scopes_mTLS.md`
- Secrets management & key management (Vault, KMS/HSM) → `15_Security_Deep_Dive_DevSecOps` (Secrets_Management_Vault_KMS.md, KMS_HSM_Key_Management.md)
- OWASP Top 10, zero trust, IAM policies, compliance (SOC2/GDPR) → `15_Security_Deep_Dive_DevSecOps`
- WAF & DDoS mitigation architecture → proposed in 15's gap list (this folder keeps request-level rate limiting/DoS basics)
- Rate-limiting algorithms & the 429 client contract → `02_API_Interface_Design/Rate_Limiting.md`
- Auth enforcement at the API edge → `02_API_Interface_Design` (Authentication.md, Authorization.md, API_Gateway.md)

## Folder housekeeping

- **Rate_Limiting_DoS_Protection.md overlaps `02_API_Interface_Design/Rate_Limiting.md`** — 02 already owns the algorithms (token bucket, distributed limiting, 429/Retry-After contract). Refocus this file purely on the security angle (brute force, volumetric vs L7 DoS, abuse detection) and cross-link 02 for mechanics, or fold it into the proposed WAF/DDoS file in 15.
- **Files are still on the legacy 16-section format** — the README promises the root TEMPLATE.md structure with the 🏋️/🎤/🎯 mastery sections, but the topic files don't have them yet; migrate as done for folders 01–05.
