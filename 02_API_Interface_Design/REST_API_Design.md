# REST API Design

## 1. Why Should I Care About This?
- **The Language of the Web**: REST (Representational State Transfer) is the default architectural style for the internet. If you build a web service, chances are it's RESTful.
- **Interoperability**: Any language (Python, JS, Go, Java) can talk to a REST API easily.
- **Scalability**: By following REST's statelessness constraint, you ensure your API can scale to millions of users without complex session management.

## 2. What Problem Does This Solve?
- **Server Coupling**: Prevents the client from needing to know about the server's internal state machine.
- **Naive Approach**: Creating custom protocols or non-standard HTTP usage (e.g., using `GET` to delete data, which can be accidentally triggered by search engine crawlers).

## 3. Core Idea (Intuition First)
REST is about **Resources (Nouns)** and **Standard Actions (Verbs)**. 
- Imagine a Library. Each book is a **Resource**. 
- You browse (`GET`), donate (`POST`), update the cover (`PUT/PATCH`), or discard (`DELETE`) it. 
- You don't need a specific "DiscardBookNow" button; you just use the standard "Discard" action on that specific "Book" location.

## 4. How It Actually Works
REST is defined by 6 architectural constraints:
1.  **Client-Server**: Separation of concerns.
2.  **Stateless**: Each request is self-contained.
3.  **Cacheable**: Responses define if they can be cached.
4.  **Uniform Interface**: Resources are identified by URIs; manipulated through representations (JSON/XML).
5.  **Layered System**: Clients can't tell if they are connected to the end server or an intermediary (Load Balancer).
6.  **Code on Demand** (Optional): Servers can temporarily extend client logic (e.g., sending JS).

## 5. Real-World Example
**GitHub API**:
- GitHub uses REST perfectly. To see your followers: `GET /users/{username}/followers`. It’s intuitive, uses standard HTTP codes, and employs "Link headers" for pagination, adhering to HATEOAS principles.

## 6. Code Example
A standard RESTful Controller for a "Task" resource:
```python
# Flask-style RESTful implementation
@app.route('/tasks/<int:task_id>', methods=['GET', 'PATCH', 'DELETE'])
def handle_task(task_id):
    if request.method == 'GET':
        return get_task(task_id), 200
    
    if request.method == 'PATCH':
        # Partial update
        update_data = request.json
        updated_task = db.update_task(task_id, update_data)
        return updated_task, 200
    
    if request.method == 'DELETE':
        db.remove_task(task_id)
        return '', 204 # No Content
```

## 7. Design Variations & Alternatives
- **Pragmatic REST**: Most APIs (like Stripe) aren't "Pure REST" (HATEOAS is often skipped), but they follow the resource/verb patterns.
- **JSON:API**: A strict specification for building REST APIs to reduce bike-shedding on payload structure.
- **Alternatives**: GraphQL (for flexible schemas) and gRPC (for high-performance internal communication).

## 8. Pros and Cons
### Pros
- **Simplicity**: No complex libraries needed; just standard HTTP.
- **Visibility**: Every proxy and Load Balancer understands HTTP verbs/codes, making monitoring easy.

### Cons
- **Over-fetching**: Getting the whole User object when you only need the name.
- **Under-fetching**: Making 3 calls to get a Post, its Author, and its Comments.

## 9. Failure Scenarios & What Can Go Wrong
- **Non-Standard Status Codes**: Returning `200 OK` with a body that says `{"error": "not found"}`. This breaks automated retry logic and monitoring.
- **Ignoring Idempotency**: `PUT` and `DELETE` should be idempotent (calling them twice has no extra effect). If they aren't, networking retries can cause data corruption.
- **Security through Obscurity**: Assuming `/admin/delete/all` is safe because it's not documented.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High. Every GET request can be cached by a CDN (like Cloudflare), reducing origin server load effectively to zero for static-ish data.
- **Cost**: High bandwidth for verbose JSON payloads compared to binary formats like Protobuf.

## 11. When NOT to Use This
- **Extremely Low-Latency App-to-App**: Use gRPC.
- **Complex, Frequently Changing Frontends**: Use GraphQL to avoid the N+1 problem and over-fetching.
- **Real-time streams**: Use WebSockets or Server-Sent Events (SSE).

## 12. Interview Perspective
- **What interviewers expect**: Clear understanding of HTTP Verbs (`PUT` vs `PATCH`), Status Codes (`401` vs `403`), and how to handle versioning (`/v1/`).
- **Common question**: "Explain the difference between idempotent and non-idempotent operations."

## 13. Connections to Other Topics
- **JSON**: The most common representation format.
- **OAuth/JWT**: How REST APIs are typically secured.

## 14. Visual Aids
- **HTTP Verb Table**: GET (Read), POST (Create), PUT (Replace), PATCH (Modify), DELETE (Remove).
- **The REST Handshake**: Client Request -> Intermediaries -> Server Response.

## 15. Summary (TL;DR)
REST is about **using the properties of HTTP** to manage application state via Resource representations.

## 16. Further Reading & Resources
- [Representational State Transfer (REST) - Roy Fielding's Dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [MDN HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP)
- [RESTful API Design Best Practices (Stack Overflow Blog)](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/)
