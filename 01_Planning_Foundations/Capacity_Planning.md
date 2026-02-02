# Capacity Planning

## 1. Why Should I Care About This?
- **Operational Death**: A system that runs out of disk space or memory doesn't just "slow down"—it crashes, often taking the database with it (corruption).
- **The Bill Shock**: Without planning, you might auto-scale into a million-dollar AWS bill overnight during a DDoS attack or a viral tweet.
- **Lead Times**: In the old days (on-prem), it took months to get new servers. Even in the cloud, spinning up 10,000 instances can hit "regional quota limits."

## 2. What Problem Does This Solve?
- **Resource Exhaustion**: Prevents "Disk Full" or "Out of Memory" errors.
- **Naive Approach**: "We'll just add more servers when it gets slow." This fails if the bottleneck is the **Database Connection Pool** or **Network Bandwidth**, which don't scale by just adding app servers.

## 3. Core Idea (Intuition First)
Capacity planning is like **buying groceries for a party**. You estimate how many guests are coming (traffic), how much they eat (CPU/RAM/Disk), and buy enough so you don't run out mid-party, but not so much that half the food rots (wasted money).

## 4. How It Actually Works
1.  **Define the Single-Node Limit**: Benchmark one server. "One m5.large can handle 500 concurrent users."
2.  **Calculate Total Requirement**: If the requirement is 100,000 users, you need `100,000 / 500 = 200 nodes`.
3.  **Add Buffer (Headroom)**: Always plan for 30-50% extra for traffic spikes.
4.  **Consider Redundancy (N+1)**: If you need 10 servers, run 11 so one can fail without impact.
5.  **Multi-Dimensionality**: Check **CPU**, **RAM**, **Disk IOPS**, and **Network Egress** independently.

## 5. Real-World Example
**Dropbox/File Storage**:
- Capacity planning for Dropbox isn't just about storage (petabytes). It's about **ingress bandwidth**. If 1 million users try to sync a file at once, the bottleneck isn't the disk space—it's the number of concurrent TCP connections the load balancer can handle.

## 6. Code Example (If Applicable)
Capacity planning often results in **Auto-scaling Policies**:
```hcl
# Infrastructure as Code (Terraform/AWS) reflecting Capacity Planning
resource "aws_autoscaling_group" "app_asg" {
  min_size             = 10  # Calculated floor
  max_size             = 100 # Calculated safety ceiling
  desired_capacity     = 20  # Based on current DAU
  
  target_tracking_configuration {
    target_value = 70.0 # Maintain 70% CPU (30% Headroom)
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
  }
}
```

## 7. Design Variations & Alternatives
- **Static Provisioning**: Pay for a fixed amount of hardware (Cheaper, but can't handle spikes).
- **Dynamic (Auto-scaling)**: Pays as you go (Expensive, handles spikes perfectly).
- **Predictive Scaling**: Use ML to scale up *before* the traffic arrives (e.g., scale up at 8 AM for a news site).

## 8. Pros and Cons
### Pros
- **Stability**: The system doesn't fall over during peak hours.
- **Cost Control**: You know exactly what your infrastructure expenditure will be.

### Cons
- **Complexity**: Hard to get right; traffic is often unpredictable.
- **Waste**: Over-estimating leads to "idle" resources that you still pay for.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Invisible" Bottleneck**: You have 90% free CPU, but your **File Descriptors** are exhausted.
- **Replication Lag**: In a DB, adding more read-replicas increases the load on the Primary to sync them, eventually killing the Primary.
- **Seasonal Spikes**: Planning for average traffic but being wiped out by Black Friday or the Super Bowl.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Proper planning ensures you have enough "shards" before the primary DB hits 10TB.
- **Reliability**: Guarantees enough replicas to survive a zone outage.
- **Cost**: Prevents over-provisioning (the #1 source of cloud waste).

## 11. When NOT to Use This
- **Pre-seed Startups**: You don't know if you'll have 10 users or 10,000. Use serverless (Lambda/Fargate) and postpone capacity planning until you find Product-Market Fit.
- **Non-Critical Background Jobs**: If a job can stay in a queue for 2 hours without issue, don't provision peak capacity for it.

## 12. Interview Perspective
- **What interviewers expect**: You to mention **Headroom** and **Utilization**. Don't say "I'll use 100 servers"; say "I'll assume each server handles X, and we need Y + 20% buffer."
- **Common questions**: "What if the database disk fills up faster than expected?"
- **Red flags**: Forgetting that data needs to be replicated (3x storage) or that indexes take up space too.

## 13. Connections to Other Topics
- **Auto Scaling**: The implementation of dynamic capacity.
- **Monitoring & Alerting**: How you know your capacity is reaching its limit.

## 14. Visual Aids
- **Step Graph**: Showing how capacity increases in discrete blocks (servers) vs. smooth traffic growth.
- **Utilization Heatmap**: Showing which resources (CPU vs RAM) are nearing limits.

## 15. Summary (TL;DR)
Capacity planning is the practice of **forecasting resource needs** to ensure availability while minimizing waste. It’s better to have it and not need it, than to need it and have your site go dark.

## 16. Further Reading & Resources
- *Site Reliability Engineering* (Google) - Chapter 18: Software Engineering in SRE.
- *Cloud Strategy* (Gregor Hohpe) - On dynamic vs fixed capacity.
- Datadog/NewRelic guides on "Capacity Monitoring."
