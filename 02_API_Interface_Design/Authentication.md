# Authentication

## 1. Why Should I Care About This?
- **The Front Door**: Authentication (AuthN) is what keeps the "bad guys" out. A flaw here means anyone could access your bank account, private messages, or company secrets.
- **User Trust**: If your site gets hacked because you stored passwords in plain text, your business reputation is finished forever.
- **Legal Compliance**: Regulations like GDPR, CCPA, and HIPAA legally mandate secure authentication and data protection.

## 2. What Problem Does This Solve?
- **Identity Verification**: Confirming that a user is who they say they are.
- **Naive Approach**: Sending `username` and `password` in the query string or storing passwords as raw strings. This allows anyone with a network sniffer or a database leak to impersonate every user.

## 3. Core Idea (Intuition First)
Authentication is like a **Passport Control**. 
- You show your documents (Username/Password/Social Login).
- The officer verifies them against their records (Database).
- Once verified, they give you a **Stamp** (Session Cookie) or a **Badge** (JWT). 
- From then on, you just show the Badge at every door; you don't have to show your passport again.

## 4. How It Actually Works
Three Main Patterns:
1.  **Session-Based**: Server stores a session ID in a DB/Redis and gives a `Cookie` to the client. Works well but hard to scale (server must store state).
2.  **Token-Based (JWT)**: Server signs a JSON object and gives it to the client. The client stores it (Local Storage). The server is **stateless** and only needs to verify the signature.
3.  **OAuth2 / OIDC**: Delegating authentication to a third party (Google, Apple, Facebook).

## 5. Real-World Example
**Google's "Sign-in with Google"**:
- Instead of every website managing passwords (which is risky), they use Google as an **Identity Provider**. The website never sees your password; they just get a "Trust Token" from Google saying "Yes, this is user X."

## 6. Code Example
Standard password hashing (never store plain text):
```python
import bcrypt

# When a user registers
def register_user(username, password):
    # Salt adds random noise to prevent rainbow table attacks
    salt = bcrypt.gensalt()
    hashed_password = bcrypt.hashpw(password.encode('utf-8'), salt)
    db.save_user(username, hashed_password)

# When a user logs in
def login(username, provided_password):
    stored_hash = db.get_password_hash(username)
    if bcrypt.checkpw(provided_password.encode('utf-8'), stored_hash):
        return generate_jwt(username) # Success
    return "Access Denied"
```

## 7. Design Variations & Alternatives
- **Multi-Factor Authentication (MFA)**: Password + SMS/TOTP/Hardware Key.
- **Biometrics**: Fingerprint/FaceID (Physical AuthN).
- **Passwordless**: Sending a "Magic Link" to an email or a one-time code to a phone.

## 8. Pros and Cons
### Pros
- **Security**: Protects sensitive data.
- **Analytics**: Knowing *who* is using your app helps with business decisions.

### Cons
- **UX Friction**: Remembering passwords and doing MFA is annoying for users.
- **Implementation Risk**: Auth is notoriously difficult to code correctly; use established libraries like **Passport.js**, **Auth0**, or **Firebase Auth**.

## 9. Failure Scenarios & What Can Go Wrong
- **Brute Force**: Allowing unlimited login attempts until a hacker guesses the password.
- **Credential Stuffing**: Hackers using leaked passwords from *other* sites to try and log into yours.
- **Weak Hashing**: Using `MD5` or `SHA-1` which are fast to crack. Use `Argon2` or `Bcrypt`.
- **JWT Storage**: Storing JWTs in `localStorage`, which is vulnerable to XSS (Cross-Site Scripting). Use `HttpOnly` cookies instead.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: JWTs make scaling easier (stateless). Session-based needs a distributed cache (Redis).
- **Reliability**: If your Auth service goes down, your *entire* app is bricked for everyone.
- **Cost**: Managed services (Okta/Auth0) are secure but can become extremely expensive as you scale.

## 11. When NOT to Use This
- **Anonymous Public Data**: Don't force users to log in just to view a news site or a public blog.
- **Internal Micro-Services**: If two services are on the same private network, they might use **mTLS** (Mutual TLS) instead of user-style authentication.

## 12. Interview Perspective
- **What interviewers expect**: Clear explanation of **Hashing vs Encryption**, why **JWT** is used in microservices, and how to handle **Password Reset** flows securely.
- **Red flags**: Saying "I will store the password in the database" without mentioning hashing/salting.

## 13. Connections to Other Topics
- **Authorization**: The step *after* authentication.
- **Encryption in Transit (TLS)**: Mandatory to protect credentials from "Man-in-the-Middle" attacks.

## 14. Visual Aids
- **OAuth 2.0 Flow**: Redirect to Google -> Consent -> Authorization Code -> Token Exchange.
- **Session vs JWT Table**: Comparing where the state is stored (Server vs Client).

## 15. Summary (TL;DR)
**Never build your own crypto.** Use industry-standard algorithms (Bcrypt/Argon2) and protocols (OAuth2/JWT) to prove a user's identity before letting them in.

## 16. Further Reading & Resources
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Auth0: JWT Introduction](https://jwt.io/introduction)
- [Okta: What is OAuth 2.0?](https://www.okta.com/intro-to-oauth2/)
- *Serious Cryptography* (Jean-Philippe Aumasson) - For deep math.
