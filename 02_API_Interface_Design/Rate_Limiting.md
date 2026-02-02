# Rate Limiting

## 1. Why Should I Care About This?
- **The "Noisy Neighbor" Problem**: One greedy user making 10,000 requests per second can starve your server of resources, making the site crawl for everyone else.
- **DDoS Mitigation**: It's your first line of defense against botnets trying to overwhelm your infrastructure.
- **SaaS Monetization**: Rate limiting is how you enforce "Free Tier" (100 req/day) and "Enterprise Tier" (Unlimited).
- **Service Stability**: Prevents "Cascading Failures" where a bug in Service A spams Service B, taking down the entire system.

## 2. What Problem Does This Solve?
- **Resource Exhaustion**: Prevents CPU, Memory, and Database connections from being saturated by a single client.
- **Naive Approach**: No rate limiting at all, or just limiting by IP address (which fails behind NAT or for mobile users on the same tower).

## 3. Core Idea (Intuition First)
Rate Limiting is like a **Subway Turnstile**. 
Even if a thousand people arrive at the station at once, they can only enter at the speed the turnstile allows. If someone tries to jump the turnstile too fast, they are blocked. This ensures the platform (server) never exceeds its safe "Fire Code" capacity.

## 4. How It Actually Works
### Key Algorithms
1.  **Token Bucket**: A bucket holds tokens. Each request takes one. Tokens are refilled at a fixed rate. Allows for "Bursts." (Most popular).
2.  **Leaky Bucket**: Requests enter a queue and are processed at a steady rate. If the queue is full, new requests are dropped. No bursts allowed.
3.  **Fixed Window Counter**: Count requests within a 1-minute window. Simple but allows "Bursts at the edges" (2x traffic at the boundary).
4.  **Sliding Window Log/Counter**: Tracks the exact timestamp of each request. Most accurate but memory-intensive.

## 5. Real-World Example
**Twitter & GitHub APIs**:
- Both provide headers like `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset`. If you exceed the limit, you get a `429 Too Many Requests` error. This allows their massive public systems to stay stable despite millions of developers hitting them.

## 6. Code Example
A simple Token Bucket implementation in Python:
```python
import time

class TokenBucket:
    def __init__(self, capacity, fill_rate):
        self.capacity = capacity # Max tokens
        self.fill_rate = fill_rate # Tokens per second
        self.tokens = capacity
        self.last_fill_time = time.time()

    def allow_request(self):
        now = time.time()
        # Refill tokens based on elapsed time
        elapsed = now - self.last_fill_time
        self.tokens = min(self.capacity, self.tokens + elapsed * self.fill_rate)
        self.last_fill_time = now

        if self.tokens >= 1:
            self.tokens -= 1
            return True
        return False

# Usage
limiter = TokenBucket(10, 1) # 10 tokens max, 1 refilled per second
if limiter.allow_request():
    process_request()
else:
    return "429 Too Many Requests"
```

## 7. Design Variations & Alternatives
- **Client-Side Throttling**: The client voluntarily slows down when it sees it's approaching a limit (essential for good citizenship).
- **Tiered Rate Limiting**: Different limits for Authenticated vs Anonymous, or Guest vs Admin.
- **Distributed Rate Limiting**: Using a central **Redis** instance so that a user is limited across all 50 of your web server nodes.

## 8. Pros and Cons
### Pros
- **Predictability**: Infrastructure load stays within known boundaries.
- **Security**: Harder to scrape data or brute-force passwords.

### Cons
- **UX Friction**: Legitimate users might be blocked during high-usage periods if the limits are too tight.
- **Complexity**: Implementing *accurate* distributed rate limiting without causing a Redis bottleneck is difficult.

## 9. Failure Scenarios & What Can Go Wrong
- **The Redis "Hot Key"**: If 1 million users are being limited by a single Redis key, Redis becomes the bottleneck. (Solution: Use **Local Caching** or **Sharded Redis**).
- **Clock Drift**: Servers with different system times can disagree on window boundaries.
- **Incorrect Status Codes**: Returning `500` instead of `429`. This causes clients to retry (thinking it's a server bug), making the load even worse.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High. It protects your core services from being DDoSed.
- **Reliability**: A mandatory component for any "highly available" system.
- **Cost**: Reduces "Autoscaling costs" by preventing waste on malicious traffic.

## 11. When NOT to Use This
- **Internal Micro-service Mesh**: In a trusted environment with strong backpressure (e.g., gRPC streams), explicit rate limiting between every service might be redundant.
- **Very Low Traffic**: Don't add a Redis dependency for a site that gets 5 visitors a day.

## 12. Interview Perspective
- **What interviewers expect**: Knowledge of **Token Bucket** vs **Leaky Bucket**. How to implement this in a **Distributed System** (Redis). Mentioning `429` status code and Retry-After headers.
- **Common question**: "How would you handle a sudden spike of 10x traffic?"

## 13. Connections to Other Topics
- **Token Bucket algorithm**: The logic behind it.
- **Redis**: The most common tool for distributed implementation.
- **DDoS Protection**: The broader security context.

## 14. Visual Aids
- **Algorithm Comparison Table**: Token Bucket vs Leaky Bucket vs Fixed Window.
- **Distributed Architecture Diagram**: Client -> LB -> App Node -> Redis (Limit Check).

## 15. Summary (TL;DR)
Rate limiting is the **governor for your API engine**. It ensures your system runs at a sustainable speed and doesn't explode when someone floors the pedal.

## 16. Further Reading & Resources
- [Stripe: Scaling your API with Rate Limiters](https://stripe.com/blog/rate-limiters)
- [Cloudflare: Rate Limiting Technical Blog](https://blog.cloudflare.com/rate-limiting-post-mortem/)
- [Google Cloud: Understanding API quotas](https://cloud.google.com/docs/quota)
- [Better Algorithms: Sliding Window Log](https://konghq.com/blog/how-to-design-a-scalable-rate-limiting-algorithm/)
