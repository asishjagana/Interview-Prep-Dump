# Health Checks & Failover

## 1. Why Should I Care About This?
- **Zero Downtime**: Even the best servers crash sometimes. Health checks are the "Eyes" of your load balancer, ensuring that if a server dies, no user ever sees an error page.
- **Auto-Recovery**: Health checks are the trigger for auto-scaling. If a server is "Unhealthy," the system can automatically kill it and start a fresh one.
- **User Trust**: Seeing a "502 Bad Gateway" once is bad; seeing it for 30 minutes while a server is dead is how you lose a company.

## 2. What Problem Does This Solve?
- **Zombie Servers**: Prevents a server that is "Running" but "Broken" (e.g., ran out of DB connections) from receiving traffic.
- **Failover**: Automatically moving traffic from a dead data center to a healthy one.
- **Naive Approach**: Thinking that if the process is "Running," it must be "Healthy."

## 3. Core Idea (Intuition First)
Health Checks are like **A Doctor checking a Pilot**.
- **Passive Monitoring**: The doctor watches the pilot fly. If he crashes, the doctor says, "He's sick." (Lags behind reality).
- **Active Health Check**: Every 5 minutes, the doctor asks, "Can you still see? Can you move your arms?" If the pilot doesn't answer or says "No," he is removed from the cockpit *before* the plane flies.
- **Failover**: Having a **Co-pilot** ready to grab the controls the instant the pilot faints.

## 4. How It Actually Works
### 1. Types of Checks
- **TCP Check**: Just check if the port is open (Layer 4). Fast but "dumb."
- **HTTP Check**: Request a specific URL (e.g., `/health`). The server returns `200 OK` only if it can talk to the database, the cache, and the disk.
- **Custom Logic**: The server says "I'm 90% full, don't send me more traffic yet" (Shedding load).

### 2. Thresholds
- **Unhealthy Threshold**: How many times in a row must a check fail before we pull the server out? (e.g., 3 fails).
- **Healthy Threshold**: How many times in a row must it succeed before we put it back in? (e.g., 2 suceeds).
- **Interval**: How often do we check? (e.g., every 5 seconds).

### 3. Failover
- **Automatic**: The LB detects the failure and updates its internal routing table instantly.
- **DNS Failover**: If the whole LB dies, DNS points to a backup LB.

## 5. Real-World Example
**The "Full Disk" Outage**:
- A server has a log file that grows until the disk is 100% full. 
- The TCP port is still open, so an L4 health check says "I'm fine!" 
- But the app can no longer write files or process requests.
- An **L7 HTTP Health Check** specifically checks `if disk_free > 5%`. It fails. The Load Balancer pulls the server out of rotation, and the SRE team gets an alert. Users never see a single failure.

## 6. Code Example
Implementing a robust health check endpoint in Node.js/Express:
```javascript
app.get('/health', async (req, res) => {
    try {
        // 1. Check Database
        await db.query('SELECT 1');
        
        // 2. Check Cache
        await redis.ping();
        
        // 3. Check Disk Space
        const disk = await checkDiskSpace();
        if (disk.free < 100 * 1024 * 1024) throw new Error("Disk Low");

        res.status(200).send({ status: 'UP', timestamp: new Date() });
    } catch (err) {
        res.status(503).send({ status: 'DOWN', error: err.message });
    }
});
```

## 7. Design Variations & Alternatives
- **Push vs Pull Health Checks**:
    - **Pull (Standard)**: LB asks server "Are you okay?"
    - **Push (Heartbeat)**: Server tells a central monitor "I'm alive!" every 2 seconds. If the monitor stops hearing from the server, it's marked dead.
- **Circuit Breaker Pattern**: The *client* stops calling a service if it keeps failing, allowing the service to recover without being hammered.

## 8. Pros and Cons
### Pros
- **High Availability**: Essential for any production system.
- **Observability**: Gives you a real-time map of your cluster's health.

### Cons
- **Load Overhead**: If you have 10 Load Balancers all checking 1,000 servers every 1 second, you are effectively DDoS-ing your own servers with health checks.
- **False Positives**: A minor network glitch causes the LB to think 50% of the cluster is dead and pull them all out, causing an actual outage (Panic Mode).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Flip-Flop"**: A server is right at the edge of memory. It passes a check, gets traffic, runs out of memory, fails a check, gets pulled, recovers, passes a check... (Solution: Increase thresholds or add a "Cool down" period).
- **Poison Pill Health Check**: The health check logic itself has a bug that crashes the server when it's called.
- **Liveness vs Readiness**: Confusion between "Is the process running?" and "Is it ready to take traffic?" (Kubernetes distinction).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves by orders of magnitude. 
- **Scalability**: Essential for dynamic scaling (knowing when a new instance is ready).

## 11. When NOT to Use This
- **Extremely Low Latency Internal Calls**: If every millisecond counts, you might use a "Fail-fast" logic instead of a "Check-first" logic.

## 12. Interview Perspective
- **What interviewers expect**: Ability to distinguish between **L4 vs L7 checks**. Understanding of **Success/Failure Thresholds**. Knowing that a health check should test *dependencies* (DB, Cache), not just the app.
- **Common question**: "How do you handle a scenario where a server is serving errors but the process is still running?"

## 13. Connections to Other Topics
- **Load Balancing Algorithms**: Algorithms use health checks to filter the destination list.
- **Circuit Breaker**: The application-level version of failure detection.
- **Auto-scaling**: Health is the feedback loop for scaling.

## 14. Visual Aids
- **The Threshold Timeline**: Fail, Fail, Fail -> Pulled; Success, Success -> Re-joined.
- **Dependency Map**: Health Check -> (DB, Redis, S3).

## 15. Summary (TL;DR)
**Health checks are regular automated probes that ensure a Load Balancer only sends traffic to functioning servers, while failover mechanisms provide a backup path when a primary system component fails.**

## 16. Further Reading & Resources
- [AWS: Target Group Health Checks](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/target-group-health-checks.html)
- [Kubernetes: Configure Liveness, Readiness and Startup Probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)
- [NGINX: HTTP Health Checks](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-checks/)
- [Martin Fowler: Circuit Breaker](https://martinfowler.com/bliki/CircuitBreaker.html)
