# Performance Metrics (Throughput, Latency, Error Rate)

## 1. Why Should I Care About This?
- **You Can't Manage What You Can't Measure**: Without metrics, saying "The site is slow" is useless. Is it slow for 1% of users or 100%? Is the DB busy or the Network? Metrics provide the evidence needed for debugging.
- **Business Revenue**: Amazon found that every 100ms of latency cost them 1% in sales. Google found a 0.5s delay in search results dropped traffic by 20%. Performance is a direct line to profit.
- **SLA Compliance**: If you promise a customer "99.9% uptime," you need precise metrics to prove you're keeping that promise.

## 2. What Problem Does This Solve?
- **Invisible Bottlenecks**: Identifying exactly which microservice or database query is causing a slowdown.
- **Alerting**: Automatically notifying engineers *before* the site crashes (e.g., "Error rate just spiked from 0.01% to 5%").
- **Naive Approach**: Looking at your local machine and saying, "It works for me."

## 3. Core Idea (Intuition First)
Performance Metrics are like **A Car's Dashboard**.
- **Latency**: The **Speedometer**. How long does it take to get from Point A to Point B? (Response time).
- **Throughput**: The **RPM / Gear**. How much work is the engine doing per second? (Requests per second).
- **Error Rate**: The **Check Engine Light**. How often is the system failing to do what it's supposed to do?

## 4. How It Actually Works
### 1. Latency (Response Time)
- Usually measured in Milliseconds (ms).
- **CRITICAL**: Don't use "Average." Use **Percentiles (P50, P90, P99)**.
    - **P99**: 99% of requests are faster than this. It represents your unluckiest users. If P99 is high, your site feels "broken" for millions of people.

### 2. Throughput (Capacity)
- Measured in Requests Per Second (RPS) or Queries Per Second (QPS).
- High throughput is good, but if it causes Latency to spike, you've hit the system's "Saturation" point.

### 3. Error Rate (Quality)
- Percentage of requests that return 5xx errors or exceptions.
- 0% is the goal, but "Three Nines" (0.1%) is reality at scale.

## 5. Real-World Example
**The 'Tail Latency' Nightmare**:
- A single frontend page makes 100 internal service calls.
- Even if the "Average" service latency is 10ms, if just *one* of those services has a P99 of 2 seconds, the entire page will wait 2 seconds. This is why P99 is the most important metric for Staff Engineers.

## 6. Code Example
Calculating P99 and Throughput using Python (Conceptual):
```python
import time
import numpy as np

response_times = []
start_time = time.time()

# Simulate 1000 requests
for _ in range(1000):
    req_start = time.time()
    result = run_query() # Some DB work
    response_times.append((time.time() - req_start) * 1000) # In ms

# CALCULATIONS
total_time = time.time() - start_time
rps = 1000 / total_time
p99 = np.percentile(response_times, 99)

print(f"Throughput: {rps:.2f} RPS")
print(f"P99 Latency: {p99:.2f} ms")
```

## 7. Design Variations & Alternatives
- **GOLDEN SIGNALS (Google)**: Latency, Traffic, Errors, and Saturation.
- **RED Metrics (Grafana)**: Rate, Errors, Duration.
- **USE Method (Infrastructure)**: Utilization, Saturation, and Errors.

## 8. Pros and Cons
### Pros
- **Objectivity**: No more guessing or "gut feelings" about performance.
- **Optimization**: You know exactly where to spend your time (e.g., "Don't fix the API, fix the DB Index").

### Cons
- **Observer Effect**: The code needed to measure performance can sometimes slow the system down (Overhead).
- **Metric Fatigue**: Collecting 1,000,000 metrics that nobody ever looks at, wasting money on monitoring storage (Datadog/New Relic bills).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Average" Trap**: Your "Average" latency is 50ms, but 10% of your users are experiencing 10 seconds. You think the site is fine while you're losing customers.
- **Skewed Percentiles**: Not having enough data points to make the percentile calculation accurate.
- **Clock Skew**: Measuring "Latency" between two servers with unsynchronized clocks can result in "Negative Latency" (an impossible physical event).

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High. You can't scale what you don't track.
- **Cost**: Monitoring can be expensive, but missing a performance issue can be 100x more expensive in lost business.

## 11. When NOT to Use This
- **Developer Toys**: Don't build a complex Prometheus dashboard for a script that only you run once a month.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain why **Percentiles** are better than **Averages**. Knowledge of the **Four Golden Signals**. Understanding of **Saturation**.
- **Common question**: "How do you know when your system needs to scale?" (Answer: Check Saturation and Latency trends).

## 13. Connections to Other Topics
- **SLA / SLO / SLI**: Metrics are the "I" (Indicators) in SLI.
- **Profiling & Benchmarking**: The "Active" way to get these metrics.

## 14. Visual Aids
- **Percentile Distribution Curve**: Showing the "Long Tail" of latency.
- **The 'Wall' Graph**: Throughput increasing until it hits a wall and Latency shoots straight up.

## 15. Summary (TL;DR)
**Performance metrics provide a data-driven view of system health through Latency (Speed), Throughput (Volume), and Error Rate (Success).**

## 16. Further Reading & Resources
- [Google Site Reliability Engineering: The Four Golden Signals](https://sre.google/sre-book/monitoring-distributed-systems/#xref_monitoring_golden-signals)
- [Prometheus: Metrics Types](https://prometheus.io/docs/concepts/metric_types/)
- [Datadog: Monitoring 101](https://www.datadoghq.com/blog/monitoring-101-collecting-data/)
- [High Scalability: Why Percentiles are Important](http://highscalability.com/blog/2016/1/11/why-averages-are-dangerous-and-what-to-use-instead.html)
