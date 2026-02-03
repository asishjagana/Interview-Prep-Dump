# High Availability (Active-Active vs Active-Passive)

## 1. Why Should I Care About This?
- **The "Impossible" Uptime**: Business-critical apps (Banks, Hospitals, AWS) cannot afford even 5 minutes of downtime. High Availability (HA) is the architecture that makes "Five Nines" (99.999% uptime) possible.
- **Disaster Recovery**: If you only have one server and it dies, your business is dead. HA ensures you always have a "Plan B" (or C, or D) ready to take over instantly.
- **Revenue Protection**: Downtime for a site like Amazon or Walmart during peak hours can cost millions of dollars *per minute*.

## 2. What Problem Does This Solve?
- **Single Point of Failure (SPOF)**: Modern systems assume that components *will* fail. HA solves the problem of keeping the system alive even when components die.
- **Maintenance Without Interruption**: Allowing you to upgrade your database or servers without stopping the app.
- **Naive Approach**: Thinking that "One really good server" is enough. It's not.

## 3. Core Idea (Intuition First)
- **Active-Passive (Hot Standby)**: Like **A Spare Tire**. You drive on 4 tires, and keep 1 in the trunk. If one tire pops, you have to stop and swap it out. The spare isn't doing anything until the primary fails.
- **Active-Active (Load Shared)**: Like **A Multi-Engine Plane**. All engines are running at the same time. If one engine dies, the plane keeps flying on the others. No stop is needed; you just lose a bit of power.

## 4. How It Actually Works
### Active-Passive
- **Primary Node**: Handles 100% of traffic.
- **Secondary Node**: Sits idle, constantly "heartbeating" the primary to check if it's alive.
- **Failover**: If Primary dies, the Load Balancer or DNS redirect traffic to the Secondary.
- **Benefit**: Simpler data consistency. The secondary is a perfect mirror.

### Active-Active
- **All Nodes**: Every node in the cluster handles traffic simultaneously.
- **Load Balancer**: Distributes 50/50 (or 33/33/33) across all nodes.
- **Benefit**: Best use of resources (you pay for both servers, you use both!). Fastest recovery (zero delay).

## 5. Real-World Example
**The "Database vs Web Server" HA**:
- **Web Servers (Active-Active)**: You have 10 web servers. They all serve traffic. If one crashes, the other 9 just work a bit harder. This is the standard for the application layer.
- **Databases (Active-Passive)**: Most SQL databases use Active-Passive. One server is the "Writer" (Primary), and the other is a "Read-only Standby." Making a database Active-Active (Multi-master) is extremely hard due to data conflicts.

## 6. Code Example
Simulating failover logic in a simple proxy (Conceptual):
```python
# ACTIVE-PASSIVE logic
def get_server():
    if server_primary.is_healthy():
        return server_primary
    elif server_secondary.is_healthy():
        return server_secondary
    else:
        raise Exception("System Offline")

# ACTIVE-ACTIVE logic
import random
def get_server_aa():
    healthy_servers = [s for s in [S1, S2, S3] if s.is_healthy()]
    if not healthy_servers:
        raise Exception("System Offline")
    return random.choice(healthy_servers) # Pick any working server
```

## 7. Design Variations & Alternatives
- **N+1 Redundancy**: Having one more server than is strictly necessary for the load.
- **Geographic HA**: Running nodes in different countries to survive a continent-level outage.
- **Quorum-based HA**: Used in distributed systems where a "Majority" (e.g., 3 out of 5) of nodes must agree for the system to be considered "Healthy."

## 8. Pros and Cons
### Active-Passive
- **Pros**: Easy data consistency, lower synchronization overhead, cheaper (standby doesn't need to be as powerful).
- **Cons**: Resources are wasted (the standby just sits there), "Failover Lag" (time taken to switch).

### Active-Active
- **Pros**: Highest availability, best performance (uses all capacity), zero failover lag.
- **Cons**: Massive complexity, data synchronization "Split Brain" risks, more expensive (must run both at full power).

## 9. Failure Scenarios & What Can Go Wrong
- **Split Brain**: In Active-Passive, both nodes think the other is dead and both try to become "Primary." They start writing different data to the disk, corrupting everything.
- **Failover Failure**: The standby server was never tested. When the primary dies, the standby tries to start but fails due to a configuration error that existed for months but was never noticed.
- **Capacity Overload (Active-Active)**: You have two servers at 80% load each. One dies. The remaining server is hit with 160% load and instantly crashes too. (Solution: Always run below 50% capacity if you only have 2 nodes).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Active-Active is the gold standard for reliability.
- **Cost**: Both models are significantly more expensive than a single node, but Active-Active makes better use of the money spent.

## 11. When NOT to Use This
- **Non-Critical Dev Environments**: Don't spend $5,000/month on 5 nodes for a "Staging" environment that only 3 developers use.
- **Static Content**: Just use a CDN; they take care of HA for you.

## 12. Interview Perspective
- **What interviewers expect**: Ability to categorize layers (e.g., "Web is usually AA, DB is usually AP"). Understanding of **Split Brain** and how to prevent it. Knowledge of **RTO (Recovery Time Objective)**.
- **Common question**: "How do you achieve 99.99% uptime for your database?" (Answer: Active-Passive replication with automatic failover).

## 13. Connections to Other Topics
- **SLA / SLO**: HA is how you meet your SLOs.
- **Database Replication**: The technology behind Active-Passive.
- **Health Checks**: The sensor that triggers the switch.

## 14. Visual Aids
- **Active-Passive Diagram**: Primary (Solid) -> Traffic; Secondary (Dashed) -> Waiting.
- **Active-Active Diagram**: Parallel lines from a Load Balancer to multiple servers.

## 15. Summary (TL;DR)
**Active-Passive uses a standby server as backup; Active-Active uses multiple servers to handle load simultaneously. Active-Active is superior for speed and availability but significantly more complex to coordinate.**

## 16. Further Reading & Resources
- [AWS: High Availability and Disaster Recovery](https://aws.amazon.com/whitepapers/high-availability-and-disaster-recovery-architectures-on-aws/)
- [RedHat: Active-active vs. active-passive clusters](https://www.redhat.com/en/topics/high-availability/active-active-vs-active-passive)
- [IBM: Highly Available architecture](https://www.ibm.com/cloud/learn/high-availability)
- [Microsoft: High availability and disaster recovery for Azure SQL](https://learn.microsoft.com/en-us/azure/azure-sql/database/high-availability-disaster-recovery-overview)
