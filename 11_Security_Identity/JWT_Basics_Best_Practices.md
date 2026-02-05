# JWT (JSON Web Tokens) Basics & Best Practices

## 1. Why Should I Care About This?
- **Stateless Scaling**: In microservices, you don't want every service querying a database to check a user's session. JWT allows the client to "carry" their own validated credentials, making the system incredibly fast and scalable.
- **Portability**: A JWT works across different languages (Java, Go, Node) and different domains (web, mobile, IoT) because it's just a standard JSON string.
- **Data Integrity**: Even though anyone can read a JWT, no one can *modify* it without being caught, thanks to digital signatures.

## 2. What Problem Does This Solve?
- **Session Management Overhead**: Eliminating the need for a central "Session Database" (like Redis) that becomes a single point of failure.
- **Cross-Service Trust**: Allowing Service A to trust a user's identity because it was signed by the Trusted Identity Provider.
- **Naive Approach**: Using plain cookies or simple API keys that have no built-in expiration or identity data.

## 3. Core Idea (Intuition First)
A JWT is like **A Digital Driver's License**.
- **The Identity**: It has your name, age, and license number written on it (**Payload**).
- **The Authenticity**: It has a holographic seal and a signature from the government (**Signature**).
- **The Usage**: When you show it to a bouncer, he doesn't call the Govt to check if it's real. He just looks at the holographic seal. If the seal is there, he trusts the data on the card.
- **The Security**: If you try to change your age from 18 to 21 with a pen, the seal will looks broken/tampered, and the bouncer will reject it.

## 4. How It Actually Works
A JWT has 3 parts separated by dots: `header.payload.signature`
### 1. Header
- Contains the algorithm used (e.g., `HS256` or `RS256`) and the type (`JWT`).

### 2. Payload (Claims)
- The actual data: `sub` (User ID), `name`, `admin: true`, and `exp` (Expiry time).
- **Careful**: This is only Base64 encoded, NOT encrypted. Anyone can read it. Never put passwords here!

### 3. Signature
- The `header` + `payload` are combined and "signed" using a secret key held by the server.

## 5. Real-World Example
**The "Microservice Hand-off"**:
1. User logs in. The **Auth Service** signs a JWT and sends it to the user.
2. User wants to "Upload a Video." They send the JWT to the **Video Service**.
3. **Video Service** looks at the signature using a "Public Key." 
4. If it matches, the service knows "This is Jane, and she has permission to upload."
5. The Video Service never had to talk to the Auth Service to verify Jane.

## 6. Code Example
Decoding a JWT in Node.js (Conceptual):
```javascript
const jwt = require('jsonwebtoken');

const token = "eyJhbGciOiJIUzI1NiIsInR... (truncated)";
const secret = "your-256-bit-secret";

try {
  // verify() checks the signature AND the expiration (exp)
  const decoded = jwt.verify(token, secret);
  console.log("Welcome back, " + decoded.name);
} catch(err) {
  console.log("Token is fake or expired!");
}
```

## 7. Design Variations & Alternatives
- **Access Tokens (JWT)**: Short-lived (e.g., 15 mins). Used for API calls.
- **Refresh Tokens (Opaque)**: Long-lived (e.g., 30 days). Used to get a new Access Token. Usually stored in a database.
- **PASETO**: A modern alternative to JWT that fixes some of JWT's historical design flaws (like the "alg: none" attack).

## 8. Pros and Cons
### Pros
- **Stateless**: No server storage needed for sessions.
- **Self-Contained**: All info is in the token.
- **Mobile Friendly**: Doesn't rely on browser-specific cookie logic.

### Cons
- **Hard to Revoke**: Once a JWT is signed, it's valid until it expires. If a user's account is stolen, you can't "Logout" that specific JWT easily. (Solution: Short expiry + Blocklists).
- **Token Size**: Putting too much data in the payload makes every HTTP request larger.
- **Zero Privacy**: Data is visible to everyone who sees the token.

## 10. Best Practices (Crucial for Interviews)
1. **Always use HTTPS**: To prevent the token from being stolen in transit.
2. **Short-lived Expiry**: Use 5-15 minute access tokens.
3. **Store Securely**: In a browser, use `HttpOnly` and `Secure` cookies to prevent XSS theft.
4. **Use Strong Algorithms**: Use `RS256` (Asymmetric) in production so that services only need the *Public Key* to verify (they can't forge tokens).
5. **Never put secrets in the payload**: No passwords, no credit card numbers.

## 9. Failure Scenarios & What Can Go Wrong
- **The "alg: none" Attack**: A hacker changes the header to say `alg: none`. Some bad libraries will accept the token without checking the signature! (Solution: Use modern, patched libraries).
- **The "Permanent Token"**: You forget to set an `exp` claim. A stolen token gives the hacker access to the user's account forever.
- **Refresh Token Theft**: If a hacker steals a 30-day refresh token, they have a 30-day window to stay logged in. (Solution: Refresh Token Rotation).

## 11. Impact on Scalability, Reliability & Cost
- **Scalability**: Massive. It's the standard for cloud-native apps.
- **Reliability**: High, but relies on your secret key staying secret.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain the **3 parts of a JWT**. Knowledge of **RS256 vs HS256**. Understanding of the **Revocation Problem**.
- **Common question**: "How do you invalidate a JWT if a user logs out?" (Answer: You can't perfectly, but you can use a Redis 'Blacklist' of logged-out IDs or just wait for short expiry).

## 13. Connections to Other Topics
- **OAuth2**: JWT is the "Access Token" format for OAuth2.
- **Encryption vs Hashing**: JWT uses Hashing/Signing, not Encryption.
- **Microservices**: JWT is the "Identity Glue."

## 14. Visual Aids
- **The 3 Colors**: Header (Red), Payload (Blue), Signature (Green).
- **The Verification Flow**: [Payload] + [Secret] = [Check Signature].

## 15. Summary (TL;DR)
**JSON Web Tokens (JWT) are a stateless, self-contained standard for securely transmitting information between parties as a JSON object, verified by a digital signature to ensure data integrity without server-side storage.**

## 16. Further Reading & Resources
- [JWT.io](https://jwt.io/) - The best place to debug and learn tokens.
- [Auth0: JWT Handbook](https://auth0.com/resources/ebooks/jwt-handbook)
- [RFC 7519: JSON Web Token Specification](https://datatracker.ietf.org/doc/html/rfc7519)
- [Okta: JWT Security Best Practices](https://developer.okta.com/blog/2020/08/07/jwt-best-practices-decoded)
