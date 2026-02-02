# API Design Fundamentals

## 1. Why Should I Care About This?
- **The "External Foundation"**: An API is a contract. Once third parties build on it, you can't easily change it without breaking their business. Bad API design is an expensive, multi-year legacy burden.
- **Developer Friction**: Confusing APIs lead to high support costs and low adoption. If an engineer can't "get" your API in 5 minutes, they will switch to a competitor.
- **System Decoupling**: Poorly defined interfaces lead to "leaky abstractions" where the internals of Service A spill into Service B, making it impossible to evolve either.

## 2. What Problem Does This Solve?
- **Inconsistency**: Prevents a system where one endpoint uses snake_case, another uses camelCase, and error formats vary wildly.
- **Naive Approach**: Exposing your database schema directly as an API. This is a disaster because every DB migration becomes a breaking API change for users.

## 3. Core Idea (Intuition First)
An API is like the **dashboard of a car**. Drivers don't need to know how the fuel injection works; they just need a steering wheel, pedals, and a speedometer that behave predictably every time they sit in a new car.

## 4. How It Actually Works
Effective API design relies on four pillars:
1.  **Uniformity**: Use consistent naming, status codes, and data structures.
2.  **Statelessness**: Every request must contain all info needed to process it (no "hidden" session state on the server).
3.  **Discovery**: High-quality documentation or self-describing formats (HATEOAS, JSON Schema).
4.  **Security-First**: Design with Authentication (Who are you?) and Authorization (What can you do?) at the entry point.

## 5. Real-World Example
**Stripe API**:
- Often cited as the "Gold Standard." They use consistent error objects, clear resource naming (`/charges`, `/customers`), and provide a "Test Mode" that behaves exactly like production. This predictability is why they dominate the payments market.

## 6. Code Example
Designing the "Resource-First" interface for a Library System:
```python
# API Layer (The "Clean" interface)
class BookAPI:
    def get_book(self, book_id: str):
        """
        Returns a simplified view of the book. 
        Hidden internals (like physical warehouse location) are NOT exposed.
        """
        raw_data = db.fetch_book_record(book_id)
        return {
            "id": raw_data.uuid,
            "title": raw_data.display_name,
            "status": "available" if raw_data.stock > 0 else "out_of_stock",
            "links": {
                "self": f"/v1/books/{raw_data.uuid}",
                "author": f"/v1/authors/{raw_data.author_id}"
            }
        }
```

## 7. Design Variations & Alternatives
- **Resource-Based (REST)**: Map verbs (GET, POST) to nouns (Books, Users).
- **Action-Based (RPC)**: Map requests to procedures (`/cancelOrder`, `/sendMessage`).
- **Graph-Based (GraphQL)**: Let the client "request" the exact shape of the data they need.

## 8. Pros and Cons
### Pros
- **Client Independence**: You can rewrite your backend in Rust without the frontend knowing.
- **Scalability**: Stateless designs are trivial to scale horizontally behind a Load Balancer.

### Cons
- **Overhead**: Every abstraction level adds slight latency.
- **Initial Cost**: Designing a *good* API takes significantly more time than just exposing a raw DB endpoint.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Fat Payload"**: Returning a 5MB JSON for a mobile app that only needs 2 fields.
- **N+1 Problem**: Forcing the client to make 100 API calls to get the details of 100 items because the list endpoint doesn't include enough data.
- **Breaking Changes**: Changing a field name from `user_id` to `account_id` without versioning.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Caching headers (`Etag`, `Cache-Control`) reduce server load by 90% for many workloads.
- **Cost**: Bad design leads to excessive bandwidth usage and higher cloud bills.

## 11. When NOT to Use This
- **Internal Helper Functions**: Don't build a full REST API for functions passing data within the same process.
- **Hyper-Performance HPC**: When every nanosecond counts (e.g., high-frequency trading), standard HTTP/JSON APIs might be too slow.

## 12. Interview Perspective
- **What interviewers expect**: Discussion on **Idempotency**, **Versioning**, and **Error Handling**. Mentioning "Return early" and "Consistent Status Codes" (e.g., 201 for Created, 400 for Bad Request).
- **Red flags**: Designing an API that is "Chatty" (requires too many round trips).

## 13. Connections to Other Topics
- **REST / gRPC / GraphQL**: The specific protocols for implementation.
- **Idempotency**: Critical for "Write" API reliability.

## 14. Visual Aids
- **The API Layer Cake**: Client -> API Gateway -> Auth -> Business Logic -> DB.
- **State Transition Diagram**: Showing how a Resource (e.g., Order) moves from `PENDING` to `SHIPPED` via API calls.

## 15. Summary (TL;DR)
An API is a **promise to the consumer**. Design it around their needs (Resources), not your internal constraints (Database Tables).

## 16. Further Reading & Resources
- [Google API Design Guide](https://cloud.google.com/apis/design)
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
- *Web API Design* (Brian Mulloy/Apigee)
- [Stripe API Documentation](https://stripe.com/docs/api)
