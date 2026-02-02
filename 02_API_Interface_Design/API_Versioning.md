# API Versioning

## 1. Why Should I Care About This?
- **The "Unbreakable Contract"**: Once an API is public, you can't just change a field type without breaking every mobile app, script, and integration built on it.
- **Iterative Growth**: You need a way to release new features and better data models without forcing existing users to upgrade immediately.
- **Reputation**: Breaking a client's production system with an unannounced API change is the fastest way to lose trust and customers.

## 2. What Problem Does This Solve?
- **Backward Compatibility**: Allows old and new versions of an API to coexist simultaneously.
- **Naive Approach**: Changing the API in-place. If user `v1` expects `name` as a string and you change it to an object `{first, last}`, their app will crash.

## 3. Core Idea (Intuition First)
API Versioning is like **Parallel Roads**. 
When the old road (v1) gets too narrow or has a bad design, you don't destroy it while people are driving on it. You build a new, better highway (v2) next to it. People can keep using the old road as long as they need, and switch to the highway when they are ready.

## 4. How It Actually Works
Three primary strategies are used:
1.  **URI Versioning**: `/v1/users`, `/v2/users`. (Most popular, easy to cache).
2.  **Header Versioning**: `Accept: application/vnd.myapi.v2+json`. (Clean URLs but harder to test in browsers).
3.  **Query Parameter Versioning**: `/users?version=2`. (Rarely used in modern systems).
4.  **Deprecation Policy**: A planned timeline for when the "Old Road" will eventually be closed.

## 5. Real-World Example
**Stripe API Versioning**:
- Stripe uses a unique approach based on **Request Dates**. Every time they change the API, they create a new "version" named after the date (e.g., `2023-10-16`). Each user is pinned to the version they started with. Stripe uses an internal **Transformation Layer** to convert current database objects back to the format the user's version expects. This is world-class engineering.

## 6. Code Example
Implementing URI-based versioning in a web framework:
```python
# Versioned Blueprints in Flask/Express style
@v1_blueprint.route('/users', methods=['GET'])
def get_users_v1():
    # Returns legacy flat format
    return {"users": ["Alice", "Bob"]}

@v2_blueprint.route('/users', methods=['GET'])
def get_users_v2():
    # Returns modern object-based format
    return {"users": [{"id": 1, "name": "Alice"}, {"id": 2, "name": "Bob"}]}

# Entry point
app.register_blueprint(v1_blueprint, url_prefix='/v1')
app.register_blueprint(v2_blueprint, url_prefix='/v2')
```

## 7. Design Variations & Alternatives
- **Point-in-Time (Date-based)**: As used by Stripe and Twilio.
- **Semantic Versioning (SemVer)**: `v1.2.3`. Usually overkill for APIs; `v1` is sufficient for breaking changes.
- **GraphQL**: Technically "versionless," as you just add new fields and mark old ones as `@deprecated`.

## 8. Pros and Cons
### Pros
- **Safety**: No risk of breaking existing integrations.
- **Flexibility**: Engineers can fix past design mistakes without fear.

### Cons
- **Maintenance Burden**: You have to maintain two (or more) sets of code and tests.
- **Technical Debt**: If you never delete `v1`, your codebase becomes bloated and confusing.

## 9. Failure Scenarios & What Can Go Wrong
- **Inconsistent Versioning**: Versioning the `/users` endpoint but forgetting to version the `/orders` endpoint that depends on the same user object.
- **Lack of Deprecation Strategy**: Having 10 versions of an API active because you are afraid to tell customers they have 12 months to upgrade.
- **Version Sprawl**: Releasing a new version for every minor fix instead of just for breaking changes.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: No direct impact, but URI versioning makes **CDN caching** much simpler (different versions have different URLs).
- **Cost**: Doubles testing and deployment complexity.

## 11. When NOT to Use This
- **Internal APIs**: If you control both the client (Frontend) and the server (Backend), you can often just synchronize a breaking change deployment instead of full versioning.
- **Early Prototypes**: Don't version until you have actual external users.

## 12. Interview Perspective
- **What interviewers expect**: Discussion on **Forward vs. Backward compatibility**, different versioning strategies (URI vs Headers), and how to handle data migration between versions.
- **Common question**: "What's the best way to handle a breaking change in a public API?"

## 13. Connections to Other Topics
- **REST API Design**: Versioning is a core component.
- **Backward Compatibility**: The goal of versioning.

## 14. Visual Aids
- **The Transition Wave**: A timeline showing users moving from v1 to v2 until v1 is retired.
- **Transformation Layer Diagram**: Showing how one DB model feeds multiple API representations.

## 15. Summary (TL;DR)
Versioning is **contract management**. Never break the contract without giving your users a clear path and a long timeline to transition.

## 16. Further Reading & Resources
- [Stripe's Blog: API Versioning at Scale](https://stripe.com/blog/api-versioning)
- [Microsoft: API Versioning Strategy](https://learn.microsoft.com/en-us/azure/architecture/best-practices/api-design#versioning-a-restful-web-api)
- [Heroku: API Design - Versioning](https://devcenter.heroku.com/articles/api-design-guidelines#versioning)
