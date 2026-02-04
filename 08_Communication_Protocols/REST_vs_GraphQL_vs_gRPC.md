# REST vs GraphQL vs gRPC

## 1. Why Should I Care About This?
- **The API Interface**: This is how you choose to "Expose" your system to the world. Picking the wrong one can lead to high latency (REST over-fetching), complexity (GraphQL), or limited client compatibility (gRPC).
- **Backend-to-Backend Speed**: gRPC is the standard for high-performance internal microservices. Knowing it is a requirement for serious Backend roles.
- **Frontend Developer Happiness**: GraphQL gives power to front-end devs, allowing them to build UI features without waiting for backend changes.

## 2. What Problem Does This Solve?
- **Over-fetching**: Getting 50 fields when you only needed the `user_name`. (Solved by GraphQL).
- **Under-fetching**: Making 5 separate API calls to get one "screen" of data. (Solved by GraphQL).
- **Performance**: High-speed, typed communication for internal systems. (Solved by gRPC).

## 3. Core Idea (Intuition First)
- **REST**: Like **A Fixed-Price Menu**. You order "Meal #5" (The Endpoint). You get exactly what's in Meal #5. If you want a different side dish, you have to buy a whole second meal.
- **GraphQL**: Like **A Subway Sandwich Shop**. You stand at the counter and point to *exactly* what you want: "I want the bread, turkey, and olives, but NO mayo." You get exactly what you asked for, and nothing else.
- **gRPC**: Like **A High-Speed Pneumatic Tube**. It's not for humans to read; it's a specialized, super-fast tube that only works between specific machines in a factory.

## 4. How It Actually Works
### 1. REST (Representational State Transfer)
- Uses **HTTP Methods** (GET, POST, PUT, DELETE).
- Stateless and resource-based (URLs).
- Standard format: JSON.
- **Key Insight**: Uses the browser's native features (Caching, Status codes) perfectly.

### 2. GraphQL (Query Language)
- Single endpoint (usually `/graphql`).
- The client sends a "Query" describing exactly which fields it wants.
- Strongly typed schema.
- **Key Insight**: Shifts the "What data do I get?" decision from the server to the client.

### 3. gRPC (Google Remote Procedure Call)
- Built on **HTTP/2**.
- Uses **Protocol Buffers (Protobuf)** instead of JSON.
- Bidirectional streaming support.
- **Key Insight**: It's "Contract First." You define the service in a `.proto` file, and it generates the client/server code for you.

## 5. Real-World Example
**The "User Dashboard" Page**:
- **REST**: Calls `/api/user`, then `/api/orders`, then `/api/recommendations`. (3 round trips, lots of useless JSON).
- **GraphQL**: Sends 1 query: `{ user { name, orders { id, date }, recommendations { product_name } } }`. (1 round trip, perfect size).
- **gRPC**: The "Orders" microservice talks to the "Inventory" microservice via gRPC. It's so fast (binary) that the services feel like they are one single program.

## 6. Code Example
Comparing a GraphQL Query to a gRPC Definition:
```graphql
# GRAPHQL QUERY (Client Side)
query GetUser($id: ID!) {
  user(id: $id) {
    name
    email
    latestOrder {
      price
    }
  }
}
```
```protobuf
// gRPC DEFINITION (Server Side)
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

message UserRequest {
  string id = 1;
}

message UserResponse {
  string name = 1;
  string email = 2;
  float latest_order_price = 3;
}
```

## 7. Design Variations & Alternatives
- **BFF (Backend for Frontend)**: A specialized REST or GraphQL API built just for a specific UI (Mobile vs Web).
- **Falcor**: Netflix's alternative to GraphQL (rarely used now).
- **tRPC**: A typesafe communication layer for TypeScript-only projects.

## 8. Pros and Cons
### REST
- **Pros**: Simple, widely understood, great browser support, easy to cache.
- **Cons**: Over-fetching, multiple round trips, no strict typing.

### GraphQL
- **Pros**: No over-fetching, self-documenting, flexible for frontend.
- **Cons**: Complex to build (Resolvers), hard to cache (everything is a POST), performance risks (N+1 queries).

### gRPC
- **Pros**: Fastest performance, strict typing, streaming support.
- **Cons**: Not easily readable by humans, poor browser support (needs a proxy like gRPC-web), strict contracts can be rigid.

## 9. Failure Scenarios & What Can Go Wrong
- **GraphQL N+1**: A client asks for "Users and their Orders." If the backend isn't optimized, it might run 1 DB query for users and 1,000 separate queries for each user's orders, crashing the DB.
- **REST Versioning Hell**: Maintaining `/v1`, `/v2`, and `/v3` of your API because you can't easily change the fixed JSON response format.
- **gRPC Proxy Lag**: Using gRPC-web adds a proxy layer that can sometimes negate the speed gains you were looking for in the first place.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: gRPC is the best for internal scaling.
- **Cost**: GraphQL reduces bandwidth costs but increases CPU costs for the server to "parse" complex queries.

## 11. When NOT to Use This
- **gRPC**: Don't use for a public-facing Web API (hard for 3rd party devs to use).
- **GraphQL**: Don't use for a very simple API with only 2-3 fields. It's over-engineering.

## 12. Interview Perspective
- **What interviewers expect**: Ability to compare all three. Understanding of **Typed schemas**. Knowledge of **Protobuf**. 
- **Common question**: "Why would you choose GraphQL over REST for a mobile app?" (Answer: Bandwidth and round-trip reduction).

## 13. Connections to Other Topics
- **Data Compression & Serialization**: gRPC uses Protobuf.
- **HTTP Versions**: gRPC requires HTTP/2.
- **Microservices**: gRPC is the "glue" of microservices.

## 14. Visual Aids
- **Comparison Table**: Caching, Typing, Payload Size, Client Complexity.
- **Flow Diagram**: Client -> GraphQL -> Many Microservices.

## 15. Summary (TL;DR)
**Use REST for simple public APIs; use GraphQL for complex, data-heavy frontends; and use gRPC for high-performance internal microservice communication.**

## 16. Further Reading & Resources
- [RESTful Web Services (O'Reilly)](https://www.oreilly.com/library/view/restful-web-services/9780596529260/)
- [Official GraphQL Documentation](https://graphql.org/learn/)
- [gRPC Guides & Documentation](https://grpc.io/docs/what-is-grpc/introduction/)
- [Apollo GraphQL Blog: REST vs GraphQL](https://www.apollographql.com/blog/graphql-vs-rest-5-reasons-why-query-languages-are-the-future/)
