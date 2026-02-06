# Disaster Recovery (RPO, RTO) & Multi-Region Setup

## 1. Why Should I Care About This?
- **Business Survival**: 40% of businesses that experience a major data disaster never reopen. If you don't have a plan for the "Nuclear Option" (a whole city goes offline), your business is a gamble.
- **The "High Nines"**: You cannot achieve 99.99% or 99.999% uptime if you only run in one region. Regions *will* fail; you must be ready to move.
- **Customer Expectations**: In the modern world, "The database is down" is not an acceptable excuse. Users expect your site to be as reliable as electricity.

## 2. What Problem Does This Solve?
- **Catastrophic Failure**: Surviving earthquakes, floods, or massive cloud provider outages.
- **Data Loss**: Ensuring that even if a server explodes, your user's data from 1 minute ago is still safe.
- **Naive Approach**: Thinking "AWS is too big to fail." (It isn't).

## 3. Core Idea (Intuition First)
Disaster Recovery (DR) is like **Having a Spare Key and a Backup Car**:
- **RPO (Recovery Point Objective)**: **How much data can you afford to lose?** If your goal is 1 hour, and the server dies at 2 PM, you must have a backup from at least 1 PM. (The "Snapshot" distance).
- **RTO (Recovery Time Objective)**: **How long can you afford to be offline?** If the server dies at 2 PM, and your RTO is 10 minutes, you must be back online by 2:10 PM. (The "Downtime" duration).
- **The Trade-off**: Lowering these numbers (Zero data loss, Zero downtime) costs **Exponentially more money**.

## 4. How It Actually Works (The 4 DR Strategies)
### 1. Backup and Restore (The "Cheapest" Way)
- You copy your data to another region every night.
- **Disaster happens**: you start new servers from scratch and load the data.
- **RPO**: 24 Hours. **RTO**: Many Hours.

### 2. Pilot Light (The "Modern" Way)
- A "Small Flame" is always burning. Your Database is replicated to the second region in real-time, but your web servers are **OFF**.
- **Disaster happens**: you turn on the web servers and point the DNS to Region 2.
- **RPO**: Minutes. **RTO**: 15-30 Minutes.

### 3. Warm Standby
- A "Small Version" of your site is always running in Region 2 (e.g., 2 servers instead of 200).
- **Disaster happens**: you scale the 2 servers up to 200 via the Load Balancer.
- **RPO**: Seconds. **RTO**: < 5 Minutes.

### 4. Multi-Site Active/Active (The "Gold Standard")
- 100% of your site is running in BOTH regions all the time.
- Users go to whichever is closer.
- **Disaster happens**: One region dies; the users just get routed to the other one. No one even notices.
- **RPO**: Zero. **RTO**: Zero.

## 5. Real-World Example
**The "GitHub" Outage (2018)**:
1. A routine maintenance task caused a database cluster in US-East to fail.
2. GitHub was offline for hours while they carefully restored data.
3. **Problem**: They had a "Backup" strategy, but the "Validation" of the backup took longer than expected.
4. **Insight**: Disaster Recovery isn't just about *having* the backup; it's about *testing* the restore process every month.

## 6. Code Example (The Failover Logic)
Conceptual Route53 (DNS) Health Check:
```hcl
# Route 53 Failover Policy
resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.primary.zone_id
  name    = "example.com"
  type    = "A"

  # If Primary is healthy, send 100% of traffic here
  failover_routing_policy {
    type = "PRIMARY"
  }

  set_identifier = "primary"
  health_check_id = aws_route53_health_check.primary_check.id
  alias { ... }
}

resource "aws_route53_record" "www_secondary" {
  # If Primary fails, Route 53 automatically switches to this
  failover_routing_policy {
    type = "SECONDARY"
  }
  ...
}
```

## 7. Design Variations & Alternatives
- **Geostatistical Routing**: Routing users based on their country.
- **Database Replication**:
    - **Synchronous**: Slowest but zero data loss.
    - **Asynchronous**: Faster but might lose 1 second of data.
- **Chaos Engineering**: Deliberately "Killing" a region once a month to prove your DR plan actually works (Netflix's **Chaos Gorilla**).

## 8. Pros and Cons
### Pros
- **Sleep at night**: No "Single Point of Failure" for the whole business.
- **Regulatory Compliance**: Required for banks and healthcare.
- **Performance**: Multi-region setups are faster for global users.

### Cons
- **Cost**: Active/Active effectively **doubles** your infrastructure bill.
- **Latency**: Syncing data between London and New York adds "Sync Lag."
- **Sync Complexity**: Handling "Write Conflicts" (User buys item in NY, user buys same item in London at the same time).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Failover loop"**: Region A is slow, so traffic moves to Region B. Region B gets overloaded and dies. Traffic moves back to Region A, killing it too. (**Cascading Failure**).
- **The "Stale DNS" Trap**: You switch the DNS, but your users' ISP has "Cached" the old IP for 24 hours. They stay on the broken site even though the new one is up.
- **The "Untested Backup"**: You have backups, but the password to the backup server was lost, or the backup file is corrupted.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: The ultimate insurance policy.
- **Cost**: The most expensive design decision you can make.

## 11. When NOT to Use This
- **Early Stage Startups**: If you hit a 1-in-a-million region failure, just apologize to your 100 users. Spend your money on features instead.
- **Non-Critical Data**: Don't build a multi-region setup for your company's "Internal Lunch Menu."

## 12. Interview Perspective
- **What interviewers expect**: Definitions of **RPO and RTO**. Knowledge of the **4 DR Strategies** (Pilot light, etc.). Understanding that **Latency is the enemy of sync**.
- **Common question**: "How do you achieve zero-data-loss cross-region replication?" (Answer: Synchronous replication. Follow-up: Note that this will slow down all writes because of the speed-of-light delay between regions).

## 13. Connections to Other Topics
- **Availability vs Consistency**: DR forces you to choose one (CAP Theorem).
- **Infrastructure as Code**: The only way to reliably "Recreate" a region in minutes.
- **Databases**: Replication is the heart of DR.

## 14. Visual Aids
- **The RPO/RTO Chart**: Time axis showing "Last Backup" and "Time to Online."
- **The 4-Strategy Cost vs Speed Chart**: Highlighting the "Pilot Light" as the best ROI.

## 15. Summary (TL;DR)
**Disaster Recovery involves defining strict RPO (Data) and RTO (Time) goals and implementing multi-region infrastructure patterns—ranging from simple backups to Active-Active architectures—to ensure business continuity during catastrophic failure.**

## 16. Further Reading & Resources
- [AWS: Disaster Recovery of Workloads on AWS](https://docs.aws.amazon.com/whitepapers/latest/disaster-recovery-workloads-on-aws/disaster-recovery-options-in-the-cloud.html)
- [Google Cloud: Disaster Recovery Planning Guide](https://cloud.google.com/architecture/disaster-recovery)
- [Microsoft: Resiliency and Disaster Recovery](https://docs.microsoft.com/en-us/azure/architecture/resiliency/index)
- [The Netflix Simian Army](https://netflix.github.io/chaosmonkey/)
