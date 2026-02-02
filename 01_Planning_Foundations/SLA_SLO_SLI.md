# SLA / SLO / SLI

## 1. Why Should I Care About This?
- **The "Reliability at any Cost" Fallacy**: Designing for 100% uptime is impossible and infinitely expensive. These metrics help you decide exactly how much "unreliability" is acceptable.
- **Conflict of Interest**: Developers want to ship features fast (risky); SREs want a stable system (safe). **Error Budgets** (derived from SLOs) provide a mathematical way to balance these goals.
- **Trust**: If you don't define what "healthy" means, you can't prove to your customers that your service is dependable.

## 2. What Problem Does This Solve?
- **Vague Goals**: Prevents saying "The system should be fast." Instead, it says "99% of requests should be < 200ms."
- **Naive Approach**: Monitoring "Everything" and alerting on "Everything." This leads to **Alert Fatigue** where engineers ignore critical failures because the pager goes off for minor blips.

## 3. Core Idea (Intuition First)
- **SLI (Indicator)**: The "Meter." (e.g., The speedometer in your car).
- **SLO (Objective)**: The "Goal." (e.g., I want to stay under 65mph).
- **SLA (Agreement)**: The "Contract." (e.g., If I get a speeding ticket, the rental company pays). This usually involves money/refunds.

## 4. How It Actually Works
1.  **Choose SLIs**: Pick the "Golden Signals" (Latency, Errors, Throughput, Saturation).
2.  **Set SLOs**: Define a target over a window (e.g., "99.9% success rate over 30 days").
3.  **Calculate Error Budget**: 
    - 99.9% uptime means you are allowed **43 minutes of downtime per month**.
    - If you use up those 43 minutes, you stop shipping features and focus on stability.
4.  **Define SLA**: A business-level promise (e.g., "99.5% uptime or we give you 10% credit back"). Note: SLA is almost always lower/looser than your internal SLO.

## 5. Real-World Example
**Google Cloud / AWS Storage (S3)**:
- **SLI**: Successful GET requests / Total GET requests.
- **SLO**: 99.9% success.
- **SLA**: If availability drops below 99.0%, customers get a 25% credit. 
- **Result**: Engineers work tirelessly to keep the internal SLO at 99.99% to ensure they never even come close to triggering the expensive SLA.

## 6. Code Example (If Applicable)
SLIs are often defined in **Prometheus/Grafana** queries:
```promql
# SLI: Availability (Success Rate)
sum(rate(http_requests_total{status=~"2.."}[5m])) 
/ 
sum(rate(http_requests_total[5m]))

# Alert: If we are burning through our Error Budget too fast
expr: (1 - (sum(rate(errors[1h])) / sum(rate(requests[1h])))) < 0.999
```

## 7. Design Variations & Alternatives
- **User-Facing SLOs**: "User can complete checkout."
- **Internal SLOs**: "Database query latency."
- **Aspirations vs. Requirements**: SLOs are hard targets; Aspirations are "nice to haves."

## 8. Pros and Cons
### Pros
- **Objectivity**: No more arguments about whether a deploy was "successful." The numbers decide.
- **Focus**: Engineers focus on the metrics that actually impact the customer experience.

### Cons
- **Gaming the System**: Engineers might optimize for "Success Rate" by returning 200 OK for partial failures.
- **Measurement Overhead**: Building the infrastructure to accurately track these metrics can be complex.

## 9. Failure Scenarios & What Can Go Wrong
- **Choosing the Wrong SLI**: Measuring CPU usage (Saturation) instead of Request Latency (User Experience). A server can be at 90% CPU but still serving requests fast.
- **Too Many SLOs**: Monitoring 50 metrics per service. Stick to the "Golden 4."
- **Setting SLOs Too High**: 100% uptime is a myth. 99.999% (5 nines) means only **5 minutes of downtime A YEAR**, which includes scheduled maintenance.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High SLOs force you to adopt **Multi-region deployments**.
- **Reliability**: Error budgets give you a "Safe space" to fail and learn.
- **Cost**: Moving from 99.9% to 99.99% usually **doubles** the infrastructure cost.

## 11. When NOT to Use This
- **Pre-Product/Market Fit**: When you are changing your features every week, don't waste time on 30-day rolling window SLOs.
- **Purely Offline Scripts**: If a data-cleaning script fails, you just run it again. No SLA needed.

## 12. Interview Perspective
- **What interviewers expect**: You to mention **Availability** (uptime) and **Durability** (database loss protection).
- **Common questions**: "How do you decide between shipping a feature and fixing a bug?" (Answer: Use the Error Budget).
- **Red flags**: Saying "The SLA will be 100%."

## 13. Connections to Other Topics
- **Observability (Metrics)**: The source data for SLIs.
- **Chaos Engineering**: Proactively testing if you can stay within your SLO during a failure.

## 14. Visual Aids
- **The "Nines" Table**: Showing uptime allowed for 99%, 99.9%, 99.99%, etc.
- **Error Budget Burn Rate Chart**: A graph showing how fast you are "spending" your allowed downtime.

## 15. Summary (TL;DR)
- **SLI is a measurement.**
- **SLO is a target.**
- **SLA is a legal contract.**
- **Error Budget is how much risk you can afford to take.**

## 16. Further Reading & Resources
- *Site Reliability Engineering* (Google) - Chapters 4 (Service Level Objectives) & 5 (Eliminating Toil).
- *The Art of Monitoring* (James Turnbull).
- SLOconf (The Service Level Objective Conference) resources.
