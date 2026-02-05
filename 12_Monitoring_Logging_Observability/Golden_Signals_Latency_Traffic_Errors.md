# Golden Signals (Latency, Traffic, Errors, Saturation)

## 1. Why Should I Care About This?
- **Standardized Visibility**: If you have 100 teams, you can't have 100 different ways of defining "Health." The Golden Signals provide a universal language for every service in your company.
- **Effective Alerting**: Instead of alerting on "CPU is 80%" (which might be fine), you alert when "Errors are > 5%." This reduces "Alert Fatigue" and keeps engineers happy.
- **Capacity Planning**: Knowing your "Saturation" helps you decide exactly when to buy more servers before the site crashes.

## 2. What Problem Does This Solve?
- **Alert Fatigue**: Stopping useless alerts that don't actually mean the user is having a bad experience.
- **The "Invisible Slowness"**: Detecting a system that is still "Up" (green) but is so slow it's effectively broken for users.
- **Naive Approach**: Monitoring "Infrastructure" (RAM/CPU) instead of "User Experience" (Latency/Errors).

## 3. Core Idea (Intuition First)
The Four Golden Signals are like **Monitoring a Busy Restaurant**:
1. **Latency (Wait Time)**: How long does it take for a customer to get their food? (10 mins? 2 hours?)
2. **Traffic (Demand)**: How many customers are currently in the restaurant? (5 people? A line out the door?)
3. **Errors (Mistakes)**: How many orders were burnt or sent to the wrong table?
4. **Saturation (Fullness)**: How many empty tables are left? Is the chef sweating and running out of ingredients?

## 4. How It Actually Works
### 1. Latency
- The time it takes to service a request.
- **Crucial Distinction**: You must track Latency for **Successful** requests separately from **Failed** requests. (An error that returns in 1ms shouldn't make your "Average Latency" look better!).
- **Metric**: Measured in milliseconds (ms) using Percentiles (P50, P95, P99).

### 2. Traffic
- A measure of how much demand is being placed on the system.
- **Metric**: Requests per second (RPS), Transactions per minute, or Bandwidth (Mbps).

### 3. Errors
- The rate of requests that fail.
- **Types**: 
    - **Explicit**: HTTP 500 errors.
    - **Implicit**: A 200 OK that contains the text "Error: Database down."
    - **Policy-based**: A request that took > 10 seconds might be considered an "Error" by your policy.

### 4. Saturation
- How "full" your service is. Most resources start to degrade in performance before they hit 100%.
- **Metric**: CPU utilization, Memory usage, Thread pool depth, Disk I/O wait.

## 5. Real-World Example
**The "Cyber Monday" Surge**:
1. **Traffic**: Your RPS jumps from 1,000 to 10,000. (Marketing is happy).
2. **Saturation**: Your DB CPU hits 90%. (SRE is worried).
3. **Latency**: Because the DB is slow, user checkout response time jumps from 200ms to 2 seconds. (User is annoyed).
4. **Errors**: The timeout logic kicks in, and 10% of users see "Service Unavailable." (Business is losing money).
*Outcome*: By watching these 4 signals, you can see the cascade and decide to "Add more DB replicas" before Step 4 happens.

## 6. Code Example
A Prometheus-style query for the Four Golden Signals:
```promql
# 1. Latency (P95)
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# 2. Traffic (RPS)
sum(rate(http_requests_total[5m]))

# 3. Error Rate (%)
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))

# 4. Saturation (CPU)
100 * (1 - avg(irate(node_cpu_seconds_total{mode="idle"}[5m])))
```

## 7. Design Variations & Alternatives
- **RED Method**: Version for Request-driven architectures (Rate, Errors, Duration).
- **USE Method**: Version for Infrastructure/Hardware (Utilization, Saturation, Errors).

## 8. Pros and Cons
### Pros
- **Focuses on the User**: Tells you if the product is actually working.
- **Actionable**: If "Saturation" is high, you scale up. If "Errors" are high, you roll back the code.
- **Consistency**: Works for a database, a load balancer, or a frontend app.

### Cons
- **Doesn't find the "Why"**: Tells you the site is slow, but doesn't tell you which 1-line bug caused it.
- **Saturation is hard to measure**: For some systems (like a Message Queue), saturation might be "Length of the line," while for others it's "IOPS."

## 9. Failure Scenarios & What Can Go Wrong
- **Tracking Averages**: If 95% of users have 100ms latency and 5% have 30 seconds, the "Average" is fine, but you are losing 5% of your customers. **Always use Percentiles (P99).**
- **Ignoring Saturation**: Some systems stay fast (Low Latency) until they hit 99% saturation, and then they suddenly explode. If you aren't monitoring saturation, you won't see the "Cliff" coming.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: The primary tool for building an SLA (Service Level Agreement).
- **Scalability**: Scaling decisions should be based on Traffic and Saturation.

## 11. When NOT to Use This
- **Batch Processing**: For a job that runs at 2 AM and takes 4 hours, "Latency" (how long a single record took) is less important than "Throughput" (how many records per second).

## 12. Interview Perspective
- **What interviewers expect**: The names of all four signals. Understanding of **Percentiles (P99)** vs Averages. Distinction between **Infrastructure signals (CPU)** and **Application signals (Errors)**.
- **Common question**: "Your service is slow. Which metrics do you check first?" (Answer: The Four Golden Signals).

## 13. Connections to Other Topics
- **SLO/SLA**: Signals are the "Data" for the SLO.
- **Load Balancing**: LBs use Traffic and Latency to make routing decisions.
- **Auto-scaling**: Scales based on Saturation signals.

## 14. Visual Aids
- **The Dashboard**: 4 charts on one screen.
- **The P99 Graph**: Showing the "Long Tail" of latency that averages miss.

## 15. Summary (TL;DR)
**The Four Golden Signals (Latency, Traffic, Errors, and Saturation) are the essential metrics required to monitor a system's health from the perspective of both the end-user and the infrastructure capacity.**

## 16. Further Reading & Resources
- [Google SRE Book: The Four Golden Signals](https://sre.google/sre-book/monitoring-distributed-systems/#intro_monitoring_golden_signals)
- [Grafana: Monitoring with the RED Method](https://grafana.com/blog/2018/08/02/the-red-method-how-to-monitor-your-services-with-prometheus/)
- [Brendan Gregg: The USE Method](https://www.brendangregg.com/usemethod.html)
- [P99 vs Average: Why it matters](https://www.dynatrace.com/news/blog/why-averages-are-dangerous-in-monitoring/)
