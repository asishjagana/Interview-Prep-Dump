# Authentication vs Authorization

## 1. Why Should I Care About This?
- **The First Line of Defense**: If you get this wrong, your system is open to the world. A bug here is a "Data Breach" headline waiting to happen.
- **Principle of Least Privilege**: Ensuring that even if someone breaks into a user's account, they can't delete your entire database.
- **Architectural Clarity**: Knowing where "Auth" happens (Gateway? Service? Sidecar?) keeps your code clean and secure.

## 2. What Problem Does This Solve?
- **Identify Theft**: Preventing someone from pretending to be another person.
- **Privilege Escalation**: Preventing a "Customer" from accessing the "Admin Dashboard."
- **Naive Approach**: Thinking that "If they can log in, they can do everything."

## 3. Core Idea (Intuition First)
- **Authentication (AuthN)**: Is about **Identity**. It answers: **"Who are you?"**
    - Like **Showing your Passport** at the airport. It proves you are who you say you are.
- **Authorization (AuthZ)**: Is about **Permission**. It answers: **"What are you allowed to do?"**
    - Like **Your Boarding Pass**. It proves you have permission to sit in Seat 4B, but NOT to enter the Cockpit.

## 4. How It Actually Works
### 1. Authentication (Who are you?)
- **Methods**:
    - **Something you know**: Password, PIN, Security Question.
    - **Something you have**: Smart card, Phone (SMS/TOTP), Hardware Key (YubiKey).
    - **Something you are**: Fingerprint, FaceID, Retina scan.
- **MFA (Multi-Factor)**: Using 2 or more of the above.

### 2. Authorization (What can you do?)
- Happens **AFTER** authentication.
- **RBAC (Role-Based)**: "Admins" can delete; "Editors" can write; "Viewers" can read.
- **ABAC (Attribute-Based)**: "Users in the UK can view this file only during office hours."

## 5. Real-World Example
**The "Corporate Office"**:
1. You arrive at the front desk and show your **Driver's License**. The guard verifies your name. (**Authentication**).
2. The guard gives you a **Keycard**. 
3. You try to open the **CEO's Office**. The card reader turns RED and stays locked. (**Authorization Failure**).
4. You try to open the **Break Room**. The card reader turns GREEN and unlocks. (**Authorization Success**).

## 6. Code Example
The mental separation in a web handler:
```javascript
async function handleUserRequest(req, res) {
  // 1. AUTHENTICATION
  const user = await AuthN.verifyToken(req.headers.authorization);
  if (!user) return res.status(401).send("Who are you?"); // 401 Unauthorized (Identity missing)

  // 2. AUTHORIZATION
  const canDelete = await AuthZ.checkPermission(user.id, 'DELETE_USER');
  if (!canDelete) return res.status(403).send("You cannot do that."); // 403 Forbidden (Identity found, but no permission)

  // 3. DO THE WORK
  return service.performAction();
}
```

## 7. Design Variations & Alternatives
- **Stateful Auth**: The server remembers you (Sessions + Cookies).
- **Stateless Auth**: The client carries their own proof (JWT).
- **Centralized Auth**: Using an Identity Provider (Okta, Auth0, Keycloak).

## 8. Pros and Cons
### Authentication
- **Pros**: Protects against 99% of random internet attacks.
- **Cons**: High friction for users (passwords are annoying), requires secure storage (salting/hashing).

### Authorization
- **Pros**: Controls the "Blast Radius" of a compromised account.
- **Cons**: Can become extremely complex in large systems (thousands of permissions).

## 9. Failure Scenarios & What Can Go Wrong
- **401 vs 403 Confusion**: Using the wrong status code. If a user isn't logged in, they should get a `401`. If they are logged in but don't have the "Admin" role, they get a `403`.
- **Insecure Direct Object Reference (IDOR)**: User A is authenticated, but they find out that changing the URL from `myapp.com/api/user/A` to `myapp.com/api/user/B` lets them see User B's data. This is an **Authorization skip**.
- **Password Fatigue**: Users choose "Password123" because it's easy to remember, defeating the AuthN layer.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: A single vulnerability can bankrupt a company (GDPR fines).
- **Scalability**: Stateless Auth (JWT) scales better than Session-based Auth because the server doesn't need to look up a session in a database for every request.

## 11. When NOT to Use This
- **Public Data**: Don't put your "Contact Us" or "Privacy Policy" page behind an Auth wall.
- **Internal Micro-microservices**: In some "Zero Trust" environments, every call is authed. But if you have two small services on a private network with no external access, simple token verification might be enough.

## 12. Interview Perspective
- **What interviewers expect**: The **401 vs 403** distinction. The **Passport vs Boarding Pass** analogy. Awareness of **RBAC vs ABAC**.
- **Common question**: "Tell me the difference between a 401 and a 403 error." (Answer: 401 = Missing/Invalid Identity; 403 = Valid Identity but insufficient permissions).

## 13. Connections to Other Topics
- **JWT**: The tool for stateless Auth.
- **API Gateway**: Usually where the first layer of AuthN happens.
- **Identity Providers (IdP)**: External tools that handle AuthN for you.

## 14. Visual Aids
- **The Toll Booth**: Showing one gate for ID (AuthN) and a second gate for the specific Route (AuthZ).
- **The Token Flow**: [User] --Credentials--> [Server] --Token--> [User].

## 15. Summary (TL;DR)
**Authentication verifies a user's identity (Who they are), while Authorization determines their permissions and access levels (What they can do).**

## 16. Further Reading & Resources
- [OWASP: Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Auth0: What is Authentication?](https://auth0.com/docs/get-started/identity-fundamentals/authentication-and-authorization)
- [Microsoft: Authentication vs. authorization](https://learn.microsoft.com/en-us/azure/active-directory/develop/authentication-vs-authorization)
- [DigitalOcean: Auth Theory](https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2)
