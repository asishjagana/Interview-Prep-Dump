# Service Level Objectives (SLO) & Error Budgets

## 1. Why Should I Care About This?
- **Ending the "Dev vs Ops" War**: Developers want to move fast (New Features!). Ops want to stay stable (Zero Changes!). SLOs provide a mathematical way for them to agree on exactly how much "Instability" is allowed.
- **Data-Driven Decisions**: Should we stop all new features and focus on bugs? With an **Error Budget**, you have a clear answer: "If we have budget left, we keep building. If the budget is zero, we fix bugs."
- **Customer Trust**: An SLO is a promise to your users. It defines what "Good" looks like for your business.

## 2. What Problem Does This Solve?
- **Unrealistic Expectations**: Preventing the "100% Uptime" trap. (100% is impossible and too expensive).
- **The "Burnout" Cycle**: Putting a stop to reckless deployments that constant trigger 3 AM pages.
- **Naive Approach**: Only having a "Feeling" that the site is slow.

## 3. Core Idea (Intuition First)
- **SLI (Indicator)**: Like **The Thermometer**. It tells you the exact number. "The patient's temperature is 101F." (Metric: 98% of requests were 200 OK).
- **SLO (Objective)**: Like **The Goal**. "The patient's temperature should stay below 100F." (Goal: 99.9% of requests should be 200 OK).
- **SLA (Agreement)**: Like **The Insurance Policy**. "If the patient's temperature stays over 100F for a week, the hospital pays you $1,000." (Legal: If we are down more than 0.1%, we give you a refund).
- **Error Budget**: Like **A Bank Account for Downtime**. If your goal is 99.9%, you are allowed 0.1% of "Errors" per month. You can "Spend" that 0.1% on risky deployments or experiments. If you spend it all, you must stop and focus on reliability.

## 4. How It Actually Works
### 1. Defining the SLI (The Math)
- `SLI = (Good Events / Total Events) * 100`
- Example: `(Requests < 200ms / Total Requests) * 100`.

### 2. Defining the SLO (The Promise)
- Common targets: "Two Nines" (99%), "Three Nines" (99.9%), "Four Nines" (99.99%).
- Every "Nine" you add makes the system **10x more expensive** and harder to build.

### 3. Managing the Error Budget
- If you have a 99.9% SLO per month, you have **43 minutes** of downtime allowed.
- Day 1: A deployment causes 10 minutes of downtime. (Budget left: 33 mins).
- Day 15: A network glitch causes 30 minutes of downtime. (Budget left: 3 mins).
- Day 16: **ALERT!** The budget is almost gone. No more risky deployments until next month.

## 5. Real-World Example
**The "Unlucky Deployment"**:
1. Team A wants to launch a new search feature.
2. The site's SLO is 99.9%. The current Error Budget is 15 minutes remaining.
3. They launch. It crashes the site for 20 minutes before they rollback.
4. **The Budget is now -5 minutes.**
5. For the rest of the month, the "Release Manager" says "NO" to all new features. The team spends 2 weeks writing tests and fixing the deployment pipeline. 
6. This ensures the site stays stable without the manager having to "Guess" or "Get Angry."

## 6. Code Example
Defining a simple Prometheus SLO calculation:
```promql
# Availability SLO: 99.9% over 30 days
# This query calculates the "Remaining Error Budget"
(0.001 - (sum(rate(http_requests_total{status=~"5.."}[30d])) / sum(rate(http_requests_total[30d])))) * 100
```

## 7. Design Variations & Alternatives
- **Burn Rate**: How fast are we staying our error budget? (High burn rate = Immediate Alarm).
- **User-centric SLO**: Measuring only the "Primary" actions. (e.g., "Buying" is more important than "Updating Profile Picture").

## 8. Pros and Cons
### Pros
- **Removes Emotion**: Decisions are based on numbers, not who is loudest in the room.
- **Empowers Developers**: If they have a large budget, they can take risks and move fast.
- **Clear Priorities**: Everyone knows exactly when "Reliability" becomes the #1 priority.

### Cons
- **Hard to Define**: It takes months to find the "Right" numbers that users actually care about.
- **Gaming the System**: If you set the SLO too low (e.g., 90%), the system is always green, but users are actually unhappy.
- **Too Many Nines**: Execs always want 99.999% until they see the price tag.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Averaged" Outage**: An SLO of 99.9% per month allows 43 minutes. If the site is down for 43 minutes in ONE block, users are furious. If it's down for 1 second every 10 minutes, users might not notice. The math is the same.
- **Ignoring the Budget**: If management keeps forcing developers to launch features even when the budget is zero, the whole framework collapses and leads to "Observability Debt."
- **Wrong Indicators**: Monitoring "CPU" as an SLO. CPU isn't a user experience. A site with 100% CPU can still be fast. A site with 1% CPU can be broken.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: The "Core Framework" of modern Site Reliability Engineering (SRE).
- **Cost**: Prevents over-engineering for 100% uptime when 99.9% is "good enough."

## 11. When NOT to Use This
- **Early Prototypes**: When you only have 10 users and are changing the entire codebase every day.
- **Internal Tools**: Fixed-cost systems where a little downtime doesn't hurt the business.

## 12. Interview Perspective
- **What interviewers expect**: The ability to distinguish between **SLI, SLO, and SLA**. Understanding of the **Error Budget** concept. Knowledge that **"100% is the wrong target."**
- **Common question**: "How do you decide when to stop shipping features and start fixing bugs?" (Answer: Use an Error Budget based on an SLO).

## 13. Connections to Other Topics
- **Monitoring (Pillars)**: SLIs are built from Metrics.
- **Golden Signals**: The best candidates for SLIs (Latency/Errors).
- **Distributed Consensus**: A tool used to achieve the "High Nines" required by strict SLOs.

## 14. Visual Aids
- **The Fuel Gauge**: An Error Budget that goes from Full (Green) to Empty (Red).
- **The Triangle**: Balanced between Speed, Cost, and Reliability.

## 15. Summary (TL;DR)
**SLIs measure performance, SLOs define targets, and SLAs set legal terms, while Error Budgets provide a data-driven mechanism to balance the conflicting goals of infrastructure reliability and feature velocity.**

## 16. Further Reading & Resources
- [Google SRE Book: Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) - The "BIBLE" of SLOs.
- [The Art of SLOs (Workbook)](https://sre.google/workbook/service-level-objectives/)
- [SLOConf](https://www.sloconf.com/) - An entire community dedicated to this topic.
- [Nobl9: SLO Platform](https://www.nobl9.com/)
