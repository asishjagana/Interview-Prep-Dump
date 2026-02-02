# Requirement Gathering

## 1. Why Should I Care About This?
- **The "Wrong Product" Risk**: Building a highly scalable system that doesn't actually solve the user's problem is the ultimate waste of engineering resources.
- **Scope Creep**: Without clear requirements, the "simple chat app" suddenly needs to support 4K video streaming and 1-million-user group calls mid-way through design.
- **Resource Allocation**: You can't decide between a SQL or NoSQL database if you don't know if the system needs strong consistency or high availability.

## 2. What Problem Does This Solve?
- **Ambiguity**: "Build Instagram" is too vague. Gathering requirements narrows it down to "Upload photos, follow people, and view a feed."
- **Naive Approach**: Starting to draw a architecture diagram before asking "Who is using this?" and "How fast must it be?". This leads to designs that are either too weak or unnecessarily expensive.

## 3. Core Idea (Intuition First)
Requirement gathering is like **defining the playing field** before the game starts. You need to know the goal (Functional) and the rules (Non-Functional/Constraints). If you don't define the rules, you're playing a game you can't win.

## 4. How It Actually Works
Divide requirements into three distinct buckets:
1.  **Functional**: What the system *does* (e.g., "User can post a tweet").
2.  **Non-Functional**: How the system *performs* (e.g., "Timeline should load in < 200ms").
3.  **Constraints/Assumptions**: The environment (e.g., "Mobile-first," "Global audience," "Limited budget").

## 5. Real-World Example
**Uber's Dispatch System**:
- **Functional**: Match a rider with a driver.
- **Non-Functional**: Latency is critical. If a rider waits 10 seconds for a "Finding driver" screen, they might switch to Lyft.
- **Constraint**: Must work on high-latency 3G networks in developing markets.

## 6. Code Example (If Applicable)
Requirements often manifest as **Service Level Objectives (SLOs)** in code/config:
```yaml
# How requirements translate to operational metrics
service_requirements:
  feature: user_feed
  latency_p99: 200ms  # Non-Functional
  availability: 99.9% # Non-Functional
  max_payload: 5MB    # Constraint
  write_consistency: eventual # Trade-off based on requirements
```

## 7. Design Variations & Alternatives
- **User-Centric**: Focus on user stories and UX journeys first.
- **Data-Centric**: Focus on the volume and shape of data (Big Data systems).
- **Business-Centric**: Focus on cost, time-to-market, and compliance (GDPR/HIPAA).

## 8. Pros and Cons
### Pros
- **Clarity**: Everyone agrees on what "success" looks like.
- **Efficiency**: Avoids building features that aren't needed.

### Cons
- **Analysis Paralysis**: Spending too much time asking questions and never starting the design.
- **Rigidity**: If requirements change (and they will), a too-rigid initial list can hinder adaptation.

## 9. Failure Scenarios & What Can Go Wrong
- **Implicit Assumptions**: Assuming the user wants "Strong Consistency" when they actually care about "High Availability."
- **Missing Scalability Targets**: Building for 1k users when the marketing team plans a 1M user launch next month.
- **Ignoring Edge Cases**: Forgetting that users might upload 2GB videos instead of 2MB photos.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High QPS requirements force the use of Load Balancers and Sharding.
- **Reliability**: 99.999% availability requirements (5 nines) drastically increase infrastructure cost and complexity.
- **Cost**: Requirements define the "Minimum Viable Infrastructure."

## 11. When NOT to Use This
- **Rapid Prototyping/R&D**: When the goal is just to see "if something is possible," extensive requirement gathering can kill innovation.
- **Simple Scripts**: One-off automation scripts don't need a formal HLD requirement phase.

## 12. Interview Perspective
- **What interviewers expect**: You to take the lead. Don't wait for them to tell you the requirements. Propose them: "I assume we need to support 100M Daily Active Users, correct?"
- **Common questions**: "What is the most important non-functional requirement here?"
- **Red flags**: Starting to draw within the first 2 minutes.

## 13. Connections to Other Topics
- **SLA/SLO/SLI**: The technical measurement of non-functional requirements.
- **Capacity Planning**: Calculating the impact of the gathered scale.

## 14. Visual Aids
- **The Requirement Pyramid**: Functional at the base, Non-Functional in the middle, Constraints at the top.
- **Venn Diagram**: Intersection of Business Needs, Technical Feasibility, and User Desires.

## 15. Summary (TL;DR)
Measure twice, cut once. Requirement gathering is about **removing uncertainty** so that every line of your design has a reason to exist.

## 16. Further Reading & Resources
- *The Mythical Man-Month* (Fred Brooks)
- *Software Requirements* (Karl Wiegers)
- Google SRE Book (Chapter on Service Level Objectives)
