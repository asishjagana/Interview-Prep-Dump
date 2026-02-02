# Constraints & Assumptions

## 1. Why Should I Care About This?
- **Infinite Options**: Without constraints, the design space is too large. Should I use a single SQL server or a globally distributed Spanner cluster?
- **Design Blind Spots**: Every design makes assumptions (e.g., "The network is reliable"). If you don't explicitly state them, they become hidden points of failure.
- **Resource Reality**: You never have infinite time, money, or hardware. Constraints force the realistic engineering choices that separate senior architects from dreamers.

## 2. What Problem Does This Solve?
- **Ambiguity Management**: Translates a broad problem into a solvable engineering task.
- **Naive Approach**: Designing a system that works "theoretically" but fails when you realize the database must run on a single machine due to license costs, or that 90% of your users are on 2G connections.

## 3. Core Idea (Intuition First)
Constraints are the **walls of the room** you are building in. Assumptions are the **foundation you're building on**. If the walls are too narrow, you can't fit the furniture (features). If the foundation is weak (bad assumptions), the whole building falls.

## 4. How It Actually Works
1.  **Identify Hard Constraints**: Non-negotiables (e.g., "Must comply with GDPR," "Must use AWS").
2.  **Define Scale Constraints**: QPS, Data Volume, Latency.
3.  **State Assumptions**: What you believe to be true but haven't proven (e.g., "Read-to-write ratio is 100:1").
4.  **Sanity Check**: Does the assumption match the real-world use case (e.g., Netflix vs. WhatsApp)?

## 5. Real-World Example
**Twitter (X)**:
- **Assumption**: Most users only read. A tiny fraction (celebrities) write.
- **Constraint**: A tweet must be delivered to millions of followers in seconds.
- **Result**: This led to a design where "fan-out" happens on write for most users, but "pull" happens on read for handles like @ElonMusk.

## 6. Code Example (If Applicable)
Constraints often lead to **Guard Rails** in code:
```python
# Enforcing constraints early
def process_upload(file_data):
    # Constraint: Max image size 10MB
    if len(file_data) > 10 * 1024 * 1024:
        raise ValueError("File exceeds limit")
    
    # Assumption: Most uploads are JPEGs
    # We optimize the fast path for JPEGs
    if file_data.starts_with(JPEG_HEADER):
        return fast_path_optimization(file_data)
    
    return standard_processing(file_data)
```

## 7. Design Variations & Alternatives
- **Conservative Pricing**: Assume 2x the predicted traffic (Safe but expensive).
- **Aggressive (Lean)**: Build for current traffic + 20% (Cheap but risky).
- **Hard vs. Soft Constraints**: Hard (Data must not be lost) vs. Soft (System should be pretty fast).

## 8. Pros and Cons
### Pros
- **Speed**: Once constraints are set, 90% of tech choices (e.g., "Should we shard?") become obvious.
- **Communication**: Stakeholders understand the boundaries of the solution.

### Cons
- **Rigidity**: If an assumption is wrong (e.g., "Users won't upload videos"), the architecture might be impossible to pivot.
- **False Precision**: Over-estimating constraints can lead to over-engineering.

## 9. Failure Scenarios & What Can Go Wrong
- **Underestimating Hotspots**: Assuming uniform data distribution when 1% of users generate 99% of traffic.
- **The "Success Disaster"**: The system works perfectly at 10k QPS but catastrophically fails at 11k because you assumed a single Redis node could handle it forever.
- **Silent Assumptions**: Assuming the Load Balancer handles SSL termination when it actually doesn't, causing high CPU on app servers.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Constraints define the sharding key.
- **Reliability**: Assumptions about "Up-time" define the number of replicas.
- **Cost**: Every constraint has a price tag. "Zero data loss" is much more expensive than "99.9% data durability."

## 11. When NOT to Use This
- **Brainstorming/Blue-Sky Research**: Don't kill a creative idea with constraints too early in the ideation phase.
- **Minimum Viable Product (MVP)**: When speed to market is the *only* constraint, ignore long-term scaling assumptions.

## 12. Interview Perspective
- **What interviewers expect**: You to ask "What are our constraints?" and "Is it okay if I assume the read-to-write ratio is X?".
- **Common questions**: "What if the storage constraint changes from 10TB to 10PB?"
- **Red flags**: Designing without asking about scale or budget.

## 13. Connections to Other Topics
- **Back-of-the-Envelope Estimation**: The math behind the constraints.
- **SLA/SLO**: Turning constraints into measurable targets.

## 14. Visual Aids
- **The Constraint Funnel**: Many possibilities at the top, a single design at the bottom.
- **Trade-off Matrix**: Showing how Constraints A and B conflict.

## 15. Summary (TL;DR)
Constraints are **enabling limitations**. They stop you from trying to build everything and force you to build something that actually works.

## 16. Further Reading & Resources
- *The Art of Scalability* (Abbott & Fisher)
- AWS Well-Architected Framework
- *Architectural Decisions Records (ADR)* - How to document assumptions.
