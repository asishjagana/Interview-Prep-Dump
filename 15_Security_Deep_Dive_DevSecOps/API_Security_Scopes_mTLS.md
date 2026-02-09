# API Security & Scopes (OAuth2, mTLS)

## 1. Why Should I Care About This?
- **The "Over-Authorized" App**: When you login to a generic cat-photo app with Google, you don't want the app to be able to "Delete all your emails." You want it to only "See your name." Scopes make this possible.
- **Machine-to-Machine Trust**: In a microservice world, Service A needs a way to prove to Service B that it is allowed to touch *only* the "Billing" data and not the "User" data.
- **Public API Exposure**: If you build a public API, your customers will demand fine-grained control over what their employees can do.

## 2. What Problem Does This Solve?
- **Granularity**: Moving from "Access is ON" to "Access is restricted to specific actions."
- **Man-in-the-middle**: Ensuring that the API caller is actually who they say they are using cryptographic certificates.
- **Naive Approach**: Thinking that a single "API Key" is enough for a complex system.

## 3. Core Idea (Intuition First)
API Scopes and mTLS are like **Valet Parking vs a Master Key**:
- **Master Key (The Old Way)**: You give the Valet your house keys, office keys, and car keys on one ring. They can drive your car, but they can also go to your house and steal your TV.
- **Valet Key (Scopes)**: You give the Valet a specific key that **only starts the car** and doesn't even open the trunk. It has a limited "Scope" of power.
- **mTLS (The ID Check)**: Not only do you give them the Valet Key, but the car **refuses to start** unless the Valet also shows their "Official Valet Employee ID" (The Certificate).

## 4. How It Actually Works
### 1. OAuth2 Scopes
- When a user logs in, they get a **JWT (Token)**.
- Inside that token is a field called `scope`: `"read:orders write:orders"`.
- The API checks this field. If the user tries to `DELETE /orders/1` but their scope only says `read`, the API returns `403 Forbidden`.

### 2. mTLS (Mutual TLS)
- In standard HTTPS, only the server shows an ID.
- In mTLS, the client *also* provides an SSL certificate.
- The server checks: "Is this certificate signed by my Trusted Authority?" This prevents "Spoofing" or "Credential Theft" because a hacker would need the physical certificate file.

## 5. Real-World Example
**The "GitHub Personal Access Token" (PAT)**:
1. You want to write a script that uploads code to GitHub.
2. You create a PAT. GitHub asks you: "Which scopes do you want?"
3. You check `repo` (access to code) but uncheck `delete_repo` (safety).
4. You run your script. Even if your script has a bug that accidentally tries to delete a repo, GitHub's API sees your restricted scope and blocks the action.
5. **Outcome**: The "Blast Radius" of a mistake or a hacked script is minimized.

## 6. Code Example (Scopes in Express)
```javascript
const express = require('express');
const { auth, requiredScopes } = require('express-oauth2-jwt-bearer');

const app = express();

// 1. Verify the Token is valid
const checkJwt = auth({
  audience: 'https://myapi.com',
  issuerBaseURL: `https://auth.example.com/`,
});

// 2. Verify the SCOPE is valid (Valet Key check)
const checkReadScope = requiredScopes('read:messages');

app.get('/api/messages', checkJwt, checkReadScope, (req, res) => {
  res.json({ message: "You have the 'read:messages' scope. Welcome!" });
});
```

## 7. Design Variations & Alternatives
- **Role-Based Access Control (RBAC)**: "Admins can do everything." (Simpler, but less granular).
- **Attribute-Based Access Control (ABAC)**: "User can edit if they own the file AND it's Tuesday." (Most complex).
- **Service Mesh (Istio)**: Handles the mTLS "Ceremony" automatically so your code doesn't have to.

## 8. Pros and Cons
### OAuth2 Scopes
- **Pros**: Standards-compliant. Excellent for user-facing apps. Very granular.
- **Cons**: Tokens can become very large. If you have 500 scopes, your HTTP headers will be massive.

### mTLS
- **Pros**: Highest level of security for microservices. No passwords to leak.
- **Cons**: Management of certificates is a major headache. If a cert expires, the service dies.

## 9. Failure Scenarios & What Can Go Wrong
- **Scoop Creep**: You have a scope called `user:all`. This is basically a Master Key. If an app gets this scope, it has total control. (Rule: **Keep scopes as small as possible**).
- **The "Broken" Middleware**: You write code to check scopes, but you put it in the wrong file or forget to apply it to a new endpoint. A hacker finds that one "Unprotected" endpoint and gets in.
- **Token Theft**: If a hacker steals a user's JWT with `write` scopes, they can do anything the user can do until the token expires. (Solution: **Short-lived tokens and Refresh tokens**).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves system "Isolation." 
- **Scalability**: Scopes are "Stateless"—the API doesn't need to check a database to know the user's permissions, it's all inside the JWT.

## 11. When NOT to Use This
- **Small monoliths**: If you only have 1 database and 1 app, standard sessions/roles are fine. 
- **Internal CLI tools**: Simple API keys might be "Good enough" for a script that only you use.

## 12. Interview Perspective
- **What interviewers expect**: The **Valet Key** analogy. Understanding that **Scopes are NOT Roles** (A role is who you are, a scope is what you can do right now). Knowledge of **JWT structure**.
- **Common question**: "How do you protect your API from a compromised microservice?" (Answer: Use mTLS for service identity and OAuth2 Scopes to limit what that service can actually call).

## 13. Connections to Other Topics
- **JWT**: The vehicle for scopes.
- **CORS**: Not security, but often confused with it.
- **Zero Trust**: Scopes are the practical implementation of "Least Privilege."

## 14. Visual Aids
- **The JWT Payload**: JSON object showing the `scope` field.
- **The Handshake**: [App] -> [Scope Request] -> [User Approval] -> [Token].

## 15. Summary (TL;DR)
**API Scopes and mTLS enhance security by enforcing fine-grained permission levels and mutual cryptographic identity verification, ensuring that even authenticated users and services can only access the minimum data necessary for their specific task.**

## 16. Further Reading & Resources
- [OAuth.net: All about Scopes](https://oauth.net/2/scope/)
- [Auth0: How Scopes Work](https://auth0.com/docs/get-started/apis/scopes)
- [Cloudflare: What is mTLS?](https://www.cloudflare.com/learning/access-management/what-is-mtls/)
- [RFC 6749: The OAuth 2.0 Authorization Framework](https://datatracker.ietf.org/doc/html/rfc6749)
