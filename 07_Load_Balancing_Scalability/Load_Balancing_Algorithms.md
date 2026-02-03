# Load Balancing Algorithms (RR, Least Conn, etc.)

## 1. Why Should I Care About This?
- **Resource Utilization**: If you have 10 servers, you want them all to work at 70% capacity. If your algorithm is bad, Server A might be at 100% (crashing) while Server B is at 10% (idle).
- **User Experience**: A bad algorithm can send a user to a "dying" server, resulting in timeouts and frustration.
- **System Stability**: Good algorithms prevent "Cascading Failures" where one overloaded server dies, pushing its load to the next server, killing it too.

## 2. What Problem Does This Solve?
- **Unbalanced Workloads**: Solving the "Heavy Hitter" problem where some requests take 10s and others 1ms.
- **Heterogeneous Hardware**: Handling a mix of "Fast" new servers and "Slow" old servers in the same cluster.
- **Naive Approach**: Always sending traffic to the first server in the list until it breaks.

## 3. Core Idea (Intuition First)
Load balancing algorithms are like **A Restaurant Hostess**.
- **Round Robin**: She walks in a circle: "Table 1, then Table 2, then Table 3..." regardless of how many people are sitting there.
- **Least Connections**: She looks at the tables and says, "Table 2 only has one person, everyone else has four. Go to Table 2."
- **IP Hash**: "Your name starts with A? You always go to Table 1." (Consistency).

## 4. How It Actually Works
### 1. Static Algorithms (Simple)
- **Round Robin**: Just cycles through the list. Best if all servers are identical and all requests are equal.
- **Weighted Round Robin**: If Server A is 2x stronger than B, it gets 2 requests for every 1 that B gets.
- **IP Hash**: Hashes the client's IP to decide the server. Good for "Sticky" behavior without complex tracking.

### 2. Dynamic Algorithms (Smarter)
- **Least Connections**: Sends requests to the server with the fewest active TCP connections. Best for long-lived connections (e.g., Database or WebSocket).
- **Least Response Time**: Pings the servers and sends traffic to the one that answers the fastest. (Calculates `Active Connections * Average Latency`).
- **Resource-Based**: The LB checks the actual CPU/RAM usage of the server before deciding.

## 5. Real-World Example
**The "Report Generation" vs "Login" traffic**:
- Imagine half your requests are "Login" (Cheap, 5ms) and half are "Download PDF Report" (Expensive, 5000ms).
- **Round Robin** will fail here. One server might get three "Report" requests at once and freeze, while another server has three "Login" requests and is bored.
- **Least Connections** is the hero. It sees the first server is "Busy" with those long-lived report connections and sends the new logins to the free servers.

## 6. Code Example
Simulating different algorithms in Python:
```python
import itertools

servers = ["S1", "S2", "S3"]

# 1. ROUND ROBIN
rr_cycle = itertools.cycle(servers)
def get_rr_server():
    return next(rr_cycle)

# 2. LEAST CONNECTIONS (Conceptual)
server_stats = {"S1": 10, "S2": 2, "S3": 15} # Active connections
def get_least_conn_server():
    return min(server_stats, key=server_stats.get)

# 3. IP HASH
def get_hash_server(client_ip):
    index = hash(client_ip) % len(servers)
    return servers[index]
```

## 7. Design Variations & Alternatives
- **Random with Two Choices**: A modern optimization used by Nginx. Pick two servers at random, then send the request to the *better* of the two. This is statistically almost as good as "Least Connections" but much faster to calculate.
- **Consistent Hashing**: (See separate file). Used for Sharding and Caching to prevent massive data movement when servers are added/removed.

## 8. Pros and Cons
### Round Robin
- **Pros**: Zero overhead, no state to track.
- **Cons**: Doesn't handle "Heavy" vs "Light" requests well.

### Least Connections
- **Pros**: Very efficient for varied workloads.
- **Cons**: Harder to track in a distributed LB setup (where multiple LBs are trying to count connections).

### IP Hash
- **Pros**: Simple "Session Affinity."
- **Cons**: If 1,000 users come from one Corporate Proxy (one IP), that one server will be overloaded.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Dead Server" Trap**: An algorithm sends traffic to a server that is "Up" (accepting connections) but is throwing "500 Internal Errors" for every request. (Solution: Pair with Health Checks).
- **Thundering Herd**: When a new, empty server is added to a "Least Connections" cluster, the LB sends *thousands* of requests to it instantly because it has "Zero" connections, causing it to crash immediately. (Solution: Use "Slow Start" or "Warm-up").

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Dynamic algorithms allow for better scaling by utilizing every byte of RAM and CPU across the cluster.
- **Reliability**: Improves by avoiding "Hot Spots" that cause crashes.

## 11. When NOT to Use This
- **Round Robin**: Don't use for WebSockets or Database connections where some sessions last 1 hour and others 1 second.
- **Least Connections**: Don't use for simple static file serving where every request is identical in cost.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain **Weighted** variants. Understanding of **Session Affinity**. Knowing why "Least Connections" is better for long-lived connections.
- **Common question**: "How do you handle a situation where some servers are more powerful than others?" (Answer: Weighted algorithms).

## 13. Connections to Other Topics
- **High Availability**: Algorithms are how you achieve HA.
- **Consistent Hashing**: The high-end version of IP Hashing.
- **Health Checks**: The requirement for any algorithm to be effective.

## 14. Visual Aids
- **Server Load Bar Charts**: RR (Uneven) vs Least Conn (Even).
- **Algorithm Decision Table**: Use case vs Recommended approach.

## 15. Summary (TL;DR)
**Load balancing algorithms distribute traffic across a cluster. Static algorithms (RR) are simple but blind, while dynamic algorithms (Least Conn) are smart but complex, aiming to equalize the load across all available resources.**

## 16. Further Reading & Resources
- [Nginx: Load Balancing Algorithms](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#methods)
- [HAProxy: Load Balancing Algorithms](https://www.haproxy.com/blog/haproxy-load-balancing-algorithms-explained/)
- [F5: Load Balancing Methods](https://www.f5.com/glossary/load-balancing-methods)
- [Envoy: Load Balancing Policies](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/load_balancers)
