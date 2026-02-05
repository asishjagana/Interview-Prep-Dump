# OAuth2 & OpenID Connect (OIDC)

## 1. Why Should I Care About This?
- **The "Login with Google" Magic**: This is how you allow users to sign up for your app in 1 click without ever seeing their password. It's better for the user (less typing) and better for you (you don't have to store passwords).
- **Security Standard**: Every major tech company in the world (Google, Facebook, Microsoft, AWS) uses these protocols. If you're building a modern API, you *must* use them.
- **Delegated Access**: Allows a third-party app (like a "Calendar Planner") to access a user's data (their "Google Calendar") without the user giving their password to the planner.

## 2. What Problem Does This Solve?
- **Password Sharing**: Preventing users from giving their "Master Password" to random third-party apps.
- **Identity as a Service**: Offloading the risk of security and password management to experts (Google/Auth0).
- **Separation of Concerns**: Splitting "Who are you?" (Identity) from "What can you do with this API?" (Access).

## 3. Core Idea (Intuition First)
- **OAuth2**: Like **A Valet Key**. When you go to a restaurant, you don't give the valet your house keys and your wallet. You give them a special key that *only* starts the car and opens the trunk. It gives them **Limited Access** to a specific resource.
- **OIDC (OpenID Connect)**: Like **The Valet's Name Tag**. OAuth2 only gives the valet the key. OIDC is a layer on top that says: "This valet is Bob, he's 25, and he works for this company." It adds **Identity Information** to the access.

## 4. How It Actually Works
### 1. The Players
1. **Resource Owner**: The User (You).
2. **Client**: The App trying to access data (e.g., "Trello").
3. **Authorization Server**: The Identity Provider (e.g., "Google").
4. **Resource Server**: Where the data lives (e.g., "Google Contacts API").

### 2. The Flow (Authorization Code Flow)
1. **Request**: App says "I want to see your contacts."
2. **Redirect**: User is sent to Google's login page.
3. **Consent**: User says "Yes, Trello can see my contacts."
4. **Code**: Google sends a temporary "Code" back to Trello.
5. **Exchange**: Trello sends that Code + a Secret to Google in the background.
6. **Token**: Google sends an **Access Token** (OAuth2) and an **ID Token** (OIDC).

## 5. Real-World Example
**Using "Spotify" with your "Facebook" friends**:
- Spotify wants to see your Facebook friend list to find people you know.
- You click "Connect to Facebook."
- You log in to **Facebook** (not Spotify).
- Facebook asks: "Allow Spotify to view your friends?"
- You click "Allow."
- Facebook gives Spotify a "Token." Spotify can now call the Facebook API for 1 hour to get the list. Spotify **NEVER** saw your Facebook password.

## 6. Code Example
The structure of an OIDC `id_token` (it's just a JWT):
```json
{
  "iss": "https://accounts.google.com",
  "sub": "123456789", 
  "aud": "my-trello-app-id",
  "iat": 1516239022,
  "exp": 1516242622,
  "name": "Jane Doe",
  "email": "jane@example.com",
  "picture": "https://avatar.com/jane.jpg"
}
```

## 7. Design Variations & Alternatives
- **Authorization Code Flow**: The standard for web apps (most secure).
- **Implicit Flow**: The "old" way for browsers (now deprecated/unsafe).
- **Client Credentials Flow**: When one server talks to another (no user involved).
- **Device Flow**: For Smart TVs that don't have a keyboard.

## 8. Pros and Cons
### Pros
- **User Trust**: Users feel safe logging in with a known entity like Google.
- **Easy MFA**: You get Google's 2FA for free.
- **Fine-grained Scopes**: You can ask for "Read-only access to files" instead of "Full power."

### Cons
- **Complexity**: Implementing the "Exchange" flow correctly is difficult.
- **Dependency**: If Google is down, your users can't log in to your app.
- **Privacy**: The Identity Provider knows exactly every time a user logs into your app.

## 9. Failure Scenarios & What Can Go Wrong
- **Secret Leakage**: You accidentally put your `client_secret` in the JavaScript code of your website. Anyone can now pretend to be your app.
- **Open Redirector**: An attacker hijacks the `redirect_uri` to send the user's secret code to their own server instead of yours.
- **Scope Creep**: An app asks for "Full Email Access" when it only needs "Email Address." This scares users away.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High. These are stateless protocols.
- **Cost**: Tools like Auth0 are expensive, but building and securing your own identity system from scratch is even more expensive in terms of developer time and risk.

## 11. When NOT to Use This
- **Extremely Simple Apps**: If you have 5 admin users, just hardcode their usernames in the environment variables.
- **Air-gapped Systems**: If your server doesn't have internet access, it can't talk to Google/Azure for verification.

## 12. Interview Perspective
- **What interviewers expect**: The difference between **Access Token** (What can I do?) and **ID Token** (Who am I?). Knowledge of the **Authorization Code Flow**. Understanding of **Scopes**.
- **Common question**: "Why is the Authorization Code Flow better than the Implicit Flow?" (Answer: It keeps the sensitive tokens in the backend, never touching the browser's URL history or local storage).

## 13. Connections to Other Topics
- **JWT**: The format used for tokens.
- **Security & Identity**: The category this belongs in.
- **SAML**: The "Enterprise/Corporate" ancestor of OIDC.

## 14. Visual Aids
- **The Flowchart Diagram**: User -> App -> Auth Server -> User -> App -> Token -> Resource.
- **The Scopes Table**: Comparing `openid`, `profile`, `email`, and custom scopes.

## 15. Summary (TL;DR)
**OAuth2 is a framework for delegated authorization (accessing data), while OpenID Connect (OIDC) is an identity layer on top of OAuth2 that provides information about the user (identity).**

## 16. Further Reading & Resources
- [OAuth.net](https://oauth.net/2/) - The official home of the protocol.
- [Okta: What is OpenID Connect?](https://developer.okta.com/docs/concepts/oauth-openid/)
- [Auth0: The Authorization Code Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow)
- [YouTube: OAuth 2.0 and OpenID Connect in plain English](https://www.youtube.com/watch?v=996OiexHze0)
