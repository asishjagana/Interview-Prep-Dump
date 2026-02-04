# API Versioning (URI vs Header vs Media Type)

## 1. Why Should I Care About This?
- **The "Breaking Change" Nightmare**: You want to update your API, but you have 10,000 users. If you change a single field name, you break 10,000 apps. Versioning is the only way to evolve your code without destroying your business.
- **Client Diversity**: Your new web app can use Version 2, while a legacy Android app from 2018 continues to use Version 1 safely.
- **Contract Management**: It defines the "Legal Agreement" between you and the developers who use your API.

## 2. What Problem Does This Solve?
- **Maintainability**: Allowing you to delete old, "bad" code while still supporting users who haven't updated.
- **Testing**: Running multiple versions of the same service in parallel.
- **Naive Approach**: Changing the API and telling users, "Sorry, you have to rewrite your app tomorrow."

## 3. Core Idea (Intuition First)
API Versioning is like **Updating a Legal Form**.
- **URI Versioning**: Like **The Form Name**. "Form-v1.pdf" and "Form-v2.pdf" are completely separate files on your computer. You pick the one you want.
- **Header Versioning**: Like **Writing the date at the top of a generic form**. The form looks the same, but the person reading it checks the date and says, "Oh, this is an old one, I'll process it differently."
- **Media Type**: Like **Specifying the Language**. "I want this form in 2024-English."

## 4. How It Actually Works
### 1. URI Versioning (Most Common)
- The version is part of the URL path.
- Example: `https://api.example.com/v1/users`
- **Pros**: Easy to see, easy to route in the load balancer (send `/v1` to the legacy cluster).
- **Cons**: Violates the idea that a URL should represent a *resource*, not a *version*.

### 2. Header Versioning (Semantic)
- The version is sent in a custom HTTP header.
- Example: `X-API-Version: 2`
- **Pros**: Keeps URLs clean. You can default to the latest version if the header is missing.
- **Cons**: Harder for developers to test in a browser URL bar.

### 3. Media Type (Accept Header)
- The version is inside the `Accept` header.
- Example: `Accept: application/vnd.example.v1+json`
- **Pros**: The most "Architecturally correct" (Content Negotiation).
- **Cons**: Highest complexity for developers to implement and for servers to parse.

## 5. Real-World Example
**The "GitHub vs Salesforce" Approach**:
- **Salesforce** uses URI Versioning: `.../services/data/v55.0/`. It is very clear, even if the URL gets very long.
- **GitHub** used to rely heavily on the `Accept` header: `application/vnd.github.v3+json`. It made their versioning very precise and "lean" but was slightly harder for new developers to understand.

## 6. Code Example
Implementing different versioning strategies in an Express backend:
```javascript
// 1. URI VERSIONING (Managed by Routing)
app.use('/v1', v1Router);
app.use('/v2', v2Router);

// 2. HEADER VERSIONING (Managed by Middleware)
app.get('/users', (req, res) => {
    const version = req.headers['x-api-version'];
    if (version === '2') {
        return res.json({ id: 1, user_full_name: "John Doe" });
    }
    // Default to V1
    res.json({ id: 1, name: "John Doe" });
});
```

## 7. Design Variations & Alternatives
- **Dot-Notation**: `v1.1`, `v1.2`. (Usually discouraged, stick to major versions like `v1`, `v2`).
- **Date-based Versioning**: (Stripe style). Users are "Pinned" to a specific date (e.g., `2023-01-16`). When the API changes, existing users keep seeing the old format until they manually "Upgrade" their account.

## 8. Pros and Cons
### URI Versioning
- **Pros**: Simple, easy debugging, clear routing.
- **Cons**: API path bloat, breaks REST purity.

### Header/Accept Versioning
- **Pros**: Clean URIs, allows for more granular "Internal" changes.
- **Cons**: Harder to cache (caching usually ignores headers), harder to debug in simple tools.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Forever Support" Trap**: You have 10 versions of your API. Your database schema has changed so much that supporting `v1` is incredibly difficult. (Solution: Define a **Sunset Policy** - "v1 will be deleted in 6 months").
- **Breaking Minor Changes**: Adding a field is safe (usually). Renaming a field is a versioned change. Deleting a field is a versioned change. Removing a Version is the ultimate breaking change.
- **Inconsistent Strategy**: Using URI for `/users` and Headers for `/payments`. This confuses developers.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves significantly by preventing regressions.
- **Cost**: High maintenance cost to keep legacy code running (Testing, Servers).

## 11. When NOT to Use This
- **Internal Only Single-App API**: If you control BOTH the app and the server, and they are always deployed together, you don't need formal versioning yet. Just update both at once.

## 12. Interview Perspective
- **What interviewers expect**: The ability to discuss the **Pros/Cons** of each approach. Understanding of **Stripe's Date-based versioning**. Knowing when a change is **"Breaking"** vs. **"Non-breaking."**
- **Common question**: "How do you handle a change where you need to move from 'first_name' and 'last_name' to a single 'full_name' field?"

## 13. Connections to Other Topics
- **REST**: Architectural constraints on versioning.
- **Load Balancing**: Routing traffic based on versions.
- **API Design**: Versioning is a core pillar of a good API.

## 14. Visual Aids
- **Comparison Table**: Discovery, Routing, Caching, Implementation.
- **The "Sunset" Timeline**: v1 (Active) -> v1 (Deprecated) -> v1 (Dead).

## 15. Summary (TL;DR)
**API versioning ensures backward compatibility as systems evolve; URI versioning is the simplest and most common, while header-based versioning provides a cleaner architectural design.**

## 16. Further Reading & Resources
- [Stripe: API Versioning Design](https://stripe.com/blog/api-versioning) - The famous "Versioning via individual transformations" guide.
- [Microsoft: API Design - Versioning](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#versioning-a-restful-web-api)
- [Versioning RESTful Services (Thoughtworks)](https://www.thoughtworks.com/insights/blog/versioning-rest-api)
- [Public API Versioning Best Practices](https://www.api-handbook.com/versioning/)
