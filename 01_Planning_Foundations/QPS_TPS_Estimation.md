# QPS / TPS Estimation

## 1. Why Should I Care About This?
- **The "Throughput" Barrier**: Every software component (Database, Load Balancer, API Framework) has a maximum number of requests it can handle per second. If you exceed it, requests "queue up," latency spikes, and the system eventually times out.
- **Concurrency vs. Throughput**: Understanding QPS helps you distinguish between "How many users are logged in" (concurrency) and "How much work the server is actually doing" (throughput).
- **Database Selection**: A system with 1 million Read QPS needs a very different database than one with 1 thousand Write TPS (Transactions).

## 2. What Problem Does This Solve?
- **Unrealistic Design**: Prevents you from assuming a single Python web server can handle 100,000 requests per second (it can't; it usually handles < 1,000 without heavy optimization).
- **Naive Approach**: Designing for "Total Daily Users" instead of "Peak Second Traffic." A system might handle 1M users a day easily, but crash at 12:00 PM when all 1M users try to check a "Daily Deal" at once.

## 3. Core Idea (Intuition First)
QPS/TPS is the **speedometer** of your system. 
- **QPS**: How many "Questions" (Reads/Simple Requests) the system answers per second.
- **TPS**: How many "Deals" (Writes/State changes) the system completes per second. Writing a check (TPS) is always slower and harder than reading a balance (QPS).

## 4. How It Actually Works
1.  **Average QPS**: `Total Daily Requests / 86,400`.
2.  **Peak QPS**: `Average QPS * (Multiplier)`. Multiplier is usually 2x to 5x (or much higher for events like Flash Sales).
3.  **Read vs. Write Ratio**: Crucial for DB choice.
    - **100:1 (Read-Heavy)**: Use aggressive caching and read-replicas.
    - **1:1 (Write-Heavy)**: Use LSM-tree databases (ScyllaDB, Cassandra) or Message Queues.
4.  **Fan-out Calculation**: One user request might trigger 10 internal service calls.
    - `User QPS = 100` -> `Internal QPS = 1,000`.

## 5. Real-World Example
**Ad-Tech Systems (Google/Facebook Ads)**:
- These systems handle **millions of QPS**. When you load a webpage, an "auction" happens in milliseconds. The estimation here isn't just about the database; it's about the **network throughput** needed to pass these tiny bits of data across the wire millions of times per second.

## 6. Code Example (If Applicable)
Load testing tools measure QPS to find the "Breaking Point":
```javascript
import http from 'k6/http';
import { sleep } from 'k6';

export const options = {
  // Goal: Find out at what QPS the latency exceeds 500ms
  vus: 100,
  duration: '30s',
};

export default function () {
  http.get('https://api.myapp.com/v1/resource');
  sleep(1); 
}
// Logic: If 100 users make 1 request/sec, we are testing 100 QPS.
```

## 7. Design Variations & Alternatives
- **Event-Based TPS**: For systems like Blockchains, TPS is the primary metric.
- **Batch Processing**: Instead of high TPS, you might have low TPS but high "Volume per Transaction."
- **Throughput vs. Goodput**: Measuring how many requests *actually succeeded* vs. how many were attempted.

## 8. Pros and Cons
### Pros
- **Precise Benchmarking**: Gives you a hard number to aim for in performance tests.
- **Bottleneck Identification**: You realize the Load Balancer can handle 50k QPS but the Auth Service only 2k.

### Cons
- **Averages Lie**: A system that averages 100 QPS but gets 10,000 in one second and 0 for the rest of the minute is still in trouble.
- **Payload Size**: 10,000 QPS of 1KB messages is vastly different from 10,000 QPS of 1MB images.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Dogpile" Effect**: High QPS hitting a cold cache, causing all requests to slam the database simultaneously.
- **Cascading Failures**: Service A handles 10k QPS, but its dependency Service B only handles 5k. When A gets a spike, B dies, which then kills A.
- **Connection Exhaustion**: High QPS causing the server to run out of ephemeral ports or socket handles.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High QPS necessitates **Stateless Services** so you can easily add more nodes.
- **Reliability**: Requires **Rate Limiting** to protect the system from being overwhelmed.
- **Cost**: Cloud providers often charge by request count (e.g., AWS Lambda, API Gateway).

## 11. When NOT to Use This
- **Low-Traffic B2B Apps**: If you only have 50 corporate clients, QPS is irrelevant. Focus on **Availability** and **Consistency**.
- **Long-Running Batch Jobs**: If a job takes 4 hours, QPS is a meaningless metric. Use **Job Completion Rate**.

## 12. Interview Perspective
- **What interviewers expect**: You to calculate Peak QPS. If DAU = 100M, and user makes 10 requests, QPS is ~12k. Say "I'll plan for 25k-50k peak."
- **Common questions**: "How does your design change if the write QPS doubles?"
- **Red flags**: Confusing QPS with "Number of Users." 1 million users doesn't mean 1 million QPS.

## 13. Connections to Other Topics
- **Rate Limiting**: The mechanism to cap QPS.
- **Load Balancer**: Distributes QPS across multiple servers.

## 14. Visual Aids
- **Throughput Curve**: Showing how latency increases as QPS approaches the system limit.
- **Read/Write Split Diagram**: Showing how 90k Read QPS goes to Cache and 1k Write TPS goes to DB.

## 15. Summary (TL;DR)
QPS/TPS measures the **workload intensity**. It tells you not just *if* your system will work, but *how fast* it has to work to stay alive.

## 16. Further Reading & Resources
- *K6 Documentation* - Best practices for load testing.
- *Little's Law* - The mathematical relationship between concurrency, throughput, and latency.
- "High Scalability" - Case studies on Pinterest/WhatsApp QPS journeys.
