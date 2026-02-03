# Load Balancer vs Reverse Proxy vs API Gateway

## 1. Why Should I Care About This?
- **Architectural Clarity**: These three terms are often used interchangeably, but they serve very different roles. Using the wrong one in a design can lead to bottlenecks or security holes.
- **Modern Microservices**: You cannot build a modern backend without at least two of these. Understanding their "Separation of Concerns" is a key Staff Engineer skill.
- **Cost Management**: Cloud providers charge differently for an "ALB" (Load Balancer) vs. "API Gateway." Knowing when to use which can save $1,000s.

## 2. What Problem Does This Solve?
- **Traffic Routing**: Getting the right request to the right service.
- **Edge Security**: Protecting your internal network from the dirty public internet.
- **Cross-Cutting Concerns**: Handling Authentication, Logging, and Rate Limiting in ONE place so you don't have to write that code in 50 different microservices.

## 3. Core Idea (Intuition First)
- **Load Balancer (The Traffic Cop)**: His only job is to stop traffic from overwhelming one server. He spreads the cars across 5 different lanes.
- **Reverse Proxy (The Intermediary)**: Like a **Receptionist**. You talk to her, she talks to the boss, and then she tells you the answer. You never actually see the boss. She hides the boss from you for safety.
- **API Gateway (The Master Orchestrator)**: Like a **Concierge + Security Guard + Translator**. He checks your ID (Auth), translates your request from Spanish to English (Protocol translation), and makes sure you don't ask too many questions (Rate Limiting).

## 4. How It Actually Works
### 1. Reverse Proxy
- Sits between the client and a *single* (or small group) of servers.
- Features: SSL Offloading, Caching, Hiding internal IP addresses.
- **Common Tools**: Nginx, Apache, Caddy.

### 2. Load Balancer
- Specifically focuses on **distributing** load across a *cluster* of servers.
- Features: Health checks, Algorithms (RR, Least Conn).
- **Common Tools**: AWS ALB, HAProxy, F5.

### 3. API Gateway
- The "Front Door" for microservices.
- Features: **Authentication**, **Rate Limiting**, **Request/Response Transformation**, **Service Discovery**.
- **Common Tools**: Kong, AWS API Gateway, Tyk, Apigee.

## 5. Real-World Example
**The "Checkout" Flow**:
1. You hit `myapp.com`. The **Load Balancer** receives the request and picks the healthiest "Gateway" node.
2. The **API Gateway** checks your JWT token. It sees you are logged in.
3. The Gateway looks at the URL `/api/v1/checkout`. It knows this belongs to the `Payments Service`.
4. The request passes through a **Reverse Proxy** inside the data center that terminates the internal SSL.
5. Finally, it hits the `Payment Microservice`.

## 6. Code Example
Comparing a simple Nginx Reverse Proxy vs a Kong API Gateway config (Conceptual):
```nginx
# NGINX (Reverse Proxy) - Simple forwarding
server {
    listen 80;
    location / {
        proxy_pass http://internal_app_server:8080;
    }
}
```
```yaml
# KONG (API Gateway) - Smart features
services:
  - name: payments-service
    url: http://payments-api:8000
    routes:
      - paths: ["/checkout"]
    plugins:
      - name: key-auth
      - name: rate-limiting
        config:
           minute: 5
```

## 7. Design Variations & Alternatives
- **Edge Load Balancing**: Putting the LB/Gateway at the CDN level (e.g., Cloudflare Workers).
- **Service Mesh**: Using a "Sidecar" proxy (like Envoy) to handle these features *between* microservices, not just at the front door.

## 8. Pros and Cons
### API Gateway
- **Pros**: Centralized control, simplifies client code, high security.
- **Cons**: Single point of failure, can become a "Gigantic Monolith" of configuration, adds extra latency.

### Load Balancer
- **Pros**: Simple, fast, reliable.
- **Cons**: No understanding of many protocols or complex business logic.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Fat Gateway"**: Engineers keep adding more and more logic (User mapping, XML to JSON conversion) to the API Gateway until it becomes as slow and buggy as a monolith.
- **Cascading Failure**: The API Gateway's "Auth Service" lookup takes 500ms. Now every single API call in the entire company is 500ms slower.
- **IP Leakage**: A misconfigured Reverse Proxy that accidentally shares the "Internal IP" of your database in an HTTP header.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: API Gateways often include "Circuit Breakers" which improve system-wide reliability.
- **Cost**: API Gateways (especially SaaS ones like AWS) can be expensive per-request. Load Balancers are usually cheaper for pure traffic.

## 11. When NOT to Use This
- **Small Apps**: If you have 1 server and 1 API, you don't need an API Gateway. An Nginx Reverse Proxy is plenty.
- **Internal Only**: Don't put a public API Gateway between two internal services that trust each other (use a Service Mesh or direct call).

## 12. Interview Perspective
- **What interviewers expect**: The ability to list **Cross-cutting concerns** (Auth, Rate Limit) for API Gateways. Understanding that an API Gateway *is* a Reverse Proxy with extra features. 
- **Common question**: "Our frontend team is complaining they have to talk to 50 different microservices. How do we fix this?" (Answer: API Gateway / BFF Pattern).

## 13. Connections to Other Topics
- **Authentication**: A primary job of the API Gateway.
- **SLA / SLO**: Gateways are often used to enforce these (throttling).
- **Rate Limiting**: (Specific implementation detail of Gateways).

## 14. Visual Aids
- **The "Front Door" Diagram**: Users -> API Gateway -> (Microservice A, B, C).
- **Layered Infrastructure**: LB (Outer) -> Gateway (Middle) -> Service (Inner).

## 15. Summary (TL;DR)
**Load Balancers distribute traffic; Reverse Proxies hide and protect servers; API Gateways orchestrate complex microservice interactions with security and logic at the edge.**

## 16. Further Reading & Resources
- [Microsoft: API Gateway pattern vs. Direct client-to-microservice communication](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/architect-microservice-container-applications/direct-client-to-microservice-communication-versus-the-api-gateway-pattern)
- [Kong: What is an API Gateway?](https://konghq.com/learning-center/api-gateway/what-is-an-api-gateway)
- [F5: Comparison of Reverse Proxy and Load Balancer](https://www.f5.com/glossary/reverse-proxy)
- [Nginx vs HAProxy vs Kong](https://stackshare.io/stackups/haproxy-vs-nginx-vs-kong)
