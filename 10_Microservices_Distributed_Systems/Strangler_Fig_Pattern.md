# Strangler Fig Pattern (Migration)

## 1. Why Should I Care About This?
- **The "Big Bang" Disaster**: 90% of companies that try to rewrite their entire system from scratch (the "Rewrite") fail. They spend 2 years and millions of dollars, yet never launch. The Strangler Fig pattern ensures you launch **continuously**.
- **Risk Mitigation**: Instead of migrating everything on a Saturday night and hoping for the best, you migrate one small feature on a Tuesday morning while everyone is awake.
- **Business Value**: You can prove the new system works and delivers value in week 1, not year 2.

## 2. What Problem Does This Solve?
- **Legacy Technical Debt**: How to modernize a "messy" old system while it's still making money.
- **Scope Creep**: Preventing the rewrite from taking forever.
- **Naive Approach**: The "Big Bang" rewrite.

## 3. Core Idea (Intuition First)
The pattern is named after the **Strangler Fig tree**.
- The tree starts its life as a tiny seed in the top branches of a giant, old tree.
- It slowly sends roots down to the ground. 
- Over the years, more and more of the "Life" of the tree happens in the new fig vines.
- Eventually, the original old tree inside simply dies and rots away, leaving a hollow (but strong) new tree in its place.
- **In Software**: You build a "Proxy" at the front. You slowly move features from the Monolith to Microservices. One day, the Monolith is empty, and you just turn it off.

## 4. How It Actually Works
### 1. The Proxy (Facade)
- Put an **API Gateway** or **Reverse Proxy** in front of your system.
- All users talk to the Proxy.

### 2. The First Feature
- Pick a non-critical, small feature (e.g., "Reset Password").
- Build it as a new Microservice.

### 3. The Redirect
- Update the Proxy: "If the URL is `/reset-password`, go to the **New Service**. For everything else, go to the **Old Monolith**."

### 4. Repeat
- Slowly move more features (Users, Orders, Shipping).
- Eventually, all URLs point to Microservices.

## 5. Real-World Example
**Modernizing an Old E-commerce Site**:
- **Month 1**: Put Nginx in front. 100% traffic goes to the 10-year-old PHP Monolith.
- **Month 2**: Build the `notification-service` in Go. Proxy sends `/api/notify` to Go. 99% still PHP.
- **Month 4**: Move the "Catalog" to a new Node.js service. Proxy sends `/api/products` to Node.
- **Month 12**: Only the "Deep Admin Panel" is left in PHP. You move it and shut down the PHP server. The site never went offline once during the year.

## 6. Code Example
Nginx configuration for a Strangler migration:
```nginx
server {
    listen 80;

    # NEW SERVICES (Strangled)
    location /api/v1/auth {
        proxy_pass http://new-auth-service:3000;
    }

    location /api/v1/billing {
        proxy_pass http://new-billing-service:4000;
    }

    # LEGACY MONOLITH (Default)
    location / {
        proxy_pass http://legacy-monolith-server:8080;
    }
}
```

## 7. Design Variations & Alternatives
- **Parallel Run**: Running both the old and new code at the same time and checking if their results match (high safety, but expensive).
- **Feature Toggles**: Using flags in the code to switch between old and new logic.

## 8. Pros and Cons
### Pros
- **Extremely Low Risk**: You can "Roll back" a single feature in seconds by changing the proxy.
- **Continuous Value**: Users get improvements every month.
- **Legacy Stays Alive**: You don't have to "Freeze" the old system; you can still fix bugs there if needed.

### Cons
- **Dual Maintenance**: For a while, you are running two systems, two databases, and two sets of infrastructure.
- **Proxy Bottleneck**: The proxy is a new critical component.
- **Database Sharing**: The hardest part is when the Monolith and Microservice both need the *same* database table. (Solution: Database CDC or synchronization).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Data Desync"**: The new service updates a user's name, but the old monolith (which still handles billing) doesn't know about the change because they are using different DBs.
- **Strangulation Stalling**: You move the "Easy" 80% with the Strangler pattern, but the "Core 20%" is so tangled that you give up. Now you are stuck with a "Distributed Monolith" forever.
- **Performance Drag**: The proxy adds a small hop.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves by isolating the migration risk.
- **Cost**: Higher cost during the transition period due to running two systems.

## 11. When NOT to Use This
- **Small Apps**: If the total rewrite takes 2 weeks, just do the rewrite. 
- **Urgent Architecture Change**: If the old system is security-compromised or crashing every hour, you might not have the time for a slow strangulation.

## 12. Interview Perspective
- **What interviewers expect**: The **Tree Analogy**. Understanding of the **Proxy/Facade**. Awareness of the **Database migration difficulty**.
- **Common question**: "How do you migrate a 10-year-old monolith to microservices without downtime?" (Answer: Strangler Fig Pattern).

## 13. Connections to Other Topics
- **API Gateway**: The tool used to implement the facade.
- **Monolith to Microservices**: The context.
- **Canary Releasing**: Related deployment strategy.

## 14. Visual Aids
- **The Tree Metaphor**: Simple sketch of a fig vine growing around a trunk.
- **The Traffic Flow**: [User] -> [Proxy] -> {New Service / Old Monolith}.

## 15. Summary (TL;DR)
**The Strangler Fig pattern enables low-risk legacy migration by incrementally replacing old functionality with new services behind a routing facade, allowing for continuous delivery and gradual system modernization.**

## 16. Further Reading & Resources
- [Martin Fowler: Strangler Fig Application](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Microsoft: Strangler Fig Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/strangler-fig)
- [Sam Newman: Monolith to Microservices](https://samnewman.io/books/monolith-to-microservices/) - Dedicated chapters on this pattern.
- [AWS: How to migrate a monolith to microservices](https://aws.amazon.com/blogs/compute/applying-the-strangler-fig-pattern-to-modernize-monoliths/)
