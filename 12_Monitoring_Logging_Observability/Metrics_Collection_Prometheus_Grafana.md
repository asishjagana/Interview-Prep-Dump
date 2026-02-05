# Metrics Collection & Time-Series DBs (Prometheus, Grafana)

## 1. Why Should I Care About This?
- **Real-Time Heartbeat**: You need to know *right now* if your CPU is at 99% or if you're getting 0 orders. Waiting for logs to process is too slow.
- **Visual Evidence**: Humans are bad at reading terminal output but great at seeing "The line is going down!" on a dashboard. Grafana makes your system status visible to the whole company.
- **Alerting Engine**: Prometheus isn't just a database; it's an "Alert Manager." It can send a Slack message or wake you up at 3 AM if a metric breaks a threshold.

## 2. What Problem Does This Solve?
- **High-Resolution Monitoring**: Seeing what's happening every 10 seconds, not just an average over an hour.
- **Querying Complexity**: Answering questions like "What was the average latency for Users in India on iOS during the last 5 minutes?"
- **Naive Approach**: Writing a script that `greps` logs every minute to count errors. This is slow, inaccurate, and heavy.

## 3. Core Idea (Intuition First)
- **Prometheus**: Like **A Robot Nurse** in a hospital. Every 10 seconds, she walks to every patient (Service), checks their heart rate and blood pressure (Metrics), and writes the numbers in a **Big Notebook** (Time-Series DB). If a number looks scary, she rings a bell (Alert).
- **Grafana**: Like **The Big Screen in the ER**. It reads the robot's notebook and turns the numbers into beautiful, colorful heart-rate monitors and charts so the doctors (Developers) can see the status across the whole room at a glance.

## 4. How It Actually Works
### 1. Prometheus (The Collector)
- **Model**: **Pull-based**. Prometheus reaches out to your services and asks for their metrics. (Comparison: Graphite/StatsD are Push-based).
- **Format**: Plain text with labels. `http_requests_total{method="post", code="200"} 1027`.
- **Query Language**: **PromQL**. A powerful language for performing math on time-series (e.g., calculating percentage of errors).

### 2. Time-Series DB (The Storage)
- Unlike a standard SQL DB, a TSDB is specialized for **Numeric data indexed by time**.
- It uses heavy compression to store billions of data points in a very small space.

### 3. Grafana (The Visualization)
- A dashboard engine that can connect to Prometheus, MySQL, CloudWatch, and 50+ other sources.
- Uses "Templates" so one dashboard can be used for 1,000 different microservices just by switching a dropdown.

## 5. Real-World Example
**Detecting a "Memory Leak"**:
1. You look at a **Grafana** chart for `Service-A Memory Usage`.
2. The line isn't flat. It's a slow "Staircase" going up over 24 hours.
3. You use **PromQL** to see that the memory never goes down, even when traffic is zero.
4. You realize you have a memory leak. 
5. You set an **Alert** in Prometheus: `If memory > 85% for 10 minutes, page the team.`
6. Now, next time, you can fix it *before* the service crashes.

## 6. Code Example
Exposing metrics in a Node.js app using `prom-client`:
```javascript
const client = require('prom-client');
const express = require('express');
const app = express();

// 1. Create a Counter metric
const requestCounter = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status']
});

app.get('/api/data', (req, res) => {
  res.send("Data!");
  // 2. Increment the counter with labels
  requestCounter.inc({ method: 'GET', route: '/api/data', status: 200 });
});

// 3. Expose the /metrics endpoint for Prometheus to Scrape
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', client.register.contentType);
  res.end(await client.register.metrics());
});
```

## 7. Design Variations & Alternatives
- **Pushgateway**: Used for short-lived "Batch jobs" that die before Prometheus can pull from them.
- **Thanos / Cortex**: Tools that add "Infinite Storage" to Prometheus so you can keep metrics for 5 years instead of 15 days.
- **VictoriaMetrics**: A modern, high-performance alternative to the Prometheus storage engine.

## 8. Pros and Cons
### Pros
- **Dynamic Discovery**: Automatically finds new servers as they are created in Kubernetes.
- **Alerting Flexibility**: Can alert on "Trends" (e.g., "Alert if disk will be full in 4 hours"), not just "Current values."
- **Community**: The standard for modern infrastructure.

### Cons
- **Limited Detail**: Only for numbers and strings. Can't show you the "Body of the error message" (use ELK for that).
- **Single Point of Failure**: A basic Prometheus setup is not "Highly Available" (though Thanos/Cortex fix this).
- **Pull Latency**: If your scrape interval is 30s, an event can happen and you won't see it on the graph for 30s.

## 9. Failure Scenarios & What Can Go Wrong
- **Metric Explosion (Cardinality Bomb)**: You accidentally add `user_id` as a label to a metric. If you have 1 million users, Prometheus creates 1 million different time-series. It runs out of RAM and crashes your monitoring system. **NEVER use highly unique data as labels.**
- **The "Missing Scrape"**: Your network is flaky, so Prometheus misses 3 scrapes. The graph shows a "Gap." Your alert doesn't fire because it thinks there is no data.
- **Stale Data**: A service dies and stops updating its metrics. Prometheus keep showing the *last* value it saw. (Solution: Use the `absent()` or `rate()` functions to detect dead services).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves MTTR (Mean Time to Recovery) significantly.
- **Cost**: Very efficient. Storing 1 million metrics costs much less than storing 1 million log lines.

## 11. When NOT to Use This
- **Long-term Archiving**: Don't use a standard Prometheus for 7-year legal data retention.
- **Transaction Logs**: Don't use metrics to track "Who bought what." Use a SQL database.

## 12. Interview Perspective
- **What interviewers expect**: The **Pull vs Push** debate. Knowledge of **PromQL**. Awareness of the **"Cardinality"** danger.
- **Common question**: "Why is Prometheus better than just sending logs?" (Answer: Cheaper, faster for numeric trends, and built-in alerting logic).

## 13. Connections to Other Topics
- **Golden Signals**: Prometheus is where you track Latency, Traffic, etc.
- **Service Discovery**: Prometheus uses Consul/Kubernetes to find what to monitor.
- **Autoscaling**: K8s HPA usually scales based on Prometheus metrics.

## 14. Visual Aids
- **The Scrape Loop**: [Prometheus] --HTTP GET--> [/metrics].
- **The Dashboard Image**: A Grafana panel with 4 colorful lines.

## 15. Summary (TL;DR)
**Prometheus provides a robust time-series database and pull-based metrics collection engine, which together with Grafana's visualization capabilities, forms the core observability stack for modern cloud-native systems.**

## 16. Further Reading & Resources
- [Prometheus.io Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana: Getting Started](https://grafana.com/docs/grafana/latest/getting-started/)
- [Robust Perception: Prometheus Blog](https://www.robustperception.io/blog)
- [PromQL Cheat Sheet](https://promlabs.com/promql-cheat-sheet/)
