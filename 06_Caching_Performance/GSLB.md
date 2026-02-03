# Global Server Load Balancing (GSLB)

## 1. Why Should I Care About This?
- **Disaster Recovery**: If an entire AWS region (e.g., `us-east-1`) goes down, GSLB can automatically redirect 100% of your traffic to a different region (e.g., `us-west-2`) in seconds.
- **Global Latency**: GSLB ensures that a user in London hits a server in Europe, not one in California.
- **Business Compliance**: Some countries require user data to stay within their borders. GSLB can route users to region-specific clusters to enforce these rules.

## 2. What Problem Does This Solve?
- **Regional Outages**: Prevents a natural disaster or power grid failure from taking your whole company offline.
- **The "World is Round" Problem**: Solving the physical latency of data traveling across the globe.
- **Naive Approach**: Having a single DNS entry pointing to one IP address. If that server or region dies, your site is gone.

## 3. Core Idea (Intuition First)
GSLB is like **A Smart GPS for the Internet**.
- Standard Load Balancing is like a **security guard** at a building entrance telling you which floor to go to.
- GSLB is like a **travel agent** telling you *which building, in which city* you should go to based on where you are currently standing. 
- If the building in NYC is on fire, the travel agent sends you to the building in London instead.

## 4. How It Actually Works
- **DNS-Based Routing**: Most GSLB happens at the DNS level. When you type `google.com`, Google's GSLB looks at your IP address and returns the IP of the data center closest to you.
- **Health Checks**: GSLB constantly "Pings" every data center. If one doesn't answer, it's removed from the DNS results.
- **Routing Policies**:
    - **Geographic**: Route based on the user's country/continent.
    - **Weight-Based**: Send 10% of traffic to a "Testing" region and 90% to "Stable."
    - **Failover**: Send all traffic to Region A. Only use Region B if A is dead.

## 5. Real-World Example
**The "Prime Day" Failover**:
- Imagine Amazon's US-East-1 region begins to struggle under the load of Prime Day. Their GSLB system detects the rising latency and "Error Rates." It automatically starts updating DNS records to send new shoppers to US-West-2 and US-East-2, effectively spreading the load across the entire continent's infrastructure.

## 6. Code Example
Simulating GSLB logic in a DNS provider's configuration (Conceptual):
```json
{
  "name": "api.myapp.com",
  "type": "GSLB",
  "policy": "GEOGRAPHIC",
  "regions": {
    "EUROPE": {
      "endpoint": "eu-central-LB.myapp.com",
      "health_check": "/health",
      "status": "UP"
    },
    "NORTH_AMERICA": {
      "endpoint": "us-east-LB.myapp.com",
      "health_check": "/health",
      "status": "UP"
    }
  },
  "default": "us-east-LB.myapp.com"
}
```

## 7. Design Variations & Alternatives
- **Anycast Routing**: A more advanced network-level trick where multiple servers share the *same* IP address, and the internet's core routers naturally send the traffic to the nearest one. (Faster than DNS, but harder to set up).
- **HTTP/3 (QUIC)**: New protocols that handle connection migration across networks better.

## 8. Pros and Cons
### Pros
- **Resilience**: Total region-level fault tolerance.
- **User Satisfaction**: Lower "Ping" times for global users.

### Cons
- **DNS Caching**: If a region dies, users who have the old IP cached in their browser might still see errors for a few minutes (TTL issues).
- **Data Synchronization**: You must ensure the database in Region B is in sync with Region A, which is extremely difficult. (See *Consistency Models*).
- **Cost**: Running multiple data centers at once is expensive.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Ping of Death"**: If your health check is too aggressive, it might kill a region that is actually fine, causing a massive unnecessary traffic migration.
- **Split Brain**: Two regions lose connection and both think they are "The only one left," leading to data diverging wildly.
- **The "Cascading Failure"**: Region A dies. GSLB sends 100% of traffic to Region B. Region B wasn't prepared for the double load, so it *also* dies. Then C dies. Your whole global infrastructure collapses like dominos.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Allows you to scale globally.
- **Reliability**: Provides the highest level of availability (Multi-region uptime).
- **Cost**: Significant increase in cloud bills and maintenance effort.

## 11. When NOT to Use This
- **Small US-Only Startup**: If all your users are in San Francisco, multi-region GSLB is expensive overkill. Use a single-region Load Balancer.

## 12. Interview Perspective
- **What interviewers expect**: Recognition that GSLB handles **Multi-Region** while standard LB handles **Multi-Server**. Discussion on **DNS TTL** problems. Understanding of **Health Checks**.
- **Common question**: "Our main AWS region just went dark in a massive solar storm. How do we keep our customers online?"

## 13. Connections to Other Topics
- **Load Balancing**: The "local" version of this topic.
- **High Availability**: The goal of GSLB.
- **CAP Theorem**: The trade-off forced when you move data across regions.

## 14. Visual Aids
- **World Map with Lines**: User in Paris -> London Node; User in Tokyo -> Tokyo Node.
- **DNS Flowchart**: User asks DNS -> GSLB decides -> User gets IP.

## 15. Summary (TL;DR)
**Global Server Load Balancing (GSLB) is a DNS-based strategy for routing users to the healthiest and closest data center on a worldwide scale, ensuring high availability and low latency.**

## 16. Further Reading & Resources
- [F5: What is Global Server Load Balancing?](https://www.f5.com/glossary/global-server-load-balancing-gslb)
- [Cloudflare: Load Balancing Documentation](https://www.cloudflare.com/learning/performance/what-is-load-balancing/)
- [NS1: How GSLB Works](https://ns1.com/resources/global-server-load-balancing)
- [GSLB on AWS Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/routing-policy.html)
