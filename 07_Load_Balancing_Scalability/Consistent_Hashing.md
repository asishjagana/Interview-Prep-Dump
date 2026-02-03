# Consistent Hashing

## 1. Why Should I Care About This?
- **Scaling without Chaos**: In a traditional "Hash Mod N" system, if you add one server to your cache (moving from 3 to 4 nodes), **75% of your data will move**. This results in a massive "Cache Miss" storm that can crash your primary database. Consistent Hashing reduces this data movement to only **1/N** (e.g., 25%).
- **Stateful Distribution**: Essential for building distributed caches (Redis/Memcached), NoSQL databases (Cassandra/Amazon Dynamo), and Load Balancers with "Sticky" requirements.
- **High Availability**: Allows your cluster to lose nodes or add nodes dynamically without resetting the entire world.

## 2. What Problem Does This Solve?
- **The "Re-hash" Problem**: Solving the inefficiency of standard hashing where adding/removing a bucket relocates almost all items.
- **Load Imbalance**: Preventing "Hot Spots" where one server gets significantly more data/traffic than others.
- **Naive Approach**: `server = hash(key) % total_servers`. This is a disaster in any dynamic environment.

## 3. Core Idea (Intuition First)
Consistent Hashing uses the **Hash Ring**.
- Imagine a giant **Circular Clock (0 to 2^32)**.
- Both **Servers** and **Data Keys** are hashed to a position on this ring.
- To find where a key belongs, you start at the key's position and walk **Clockwise** until you hit the first server. That server "owns" the data.
- **Adding a Server**: Only the data *between* the new server and the previous server needs to move. Everything else stays exactly where it was.

## 4. How It Actually Works
### 1. The Ring
- Every server $S_i$ is mapped to a position $P_i = hash(S_i)$.
- Every key $K_j$ is mapped to a position $p_j = hash(K_j)$.
- The server for $K_j$ is the first $S_i$ where $P_i \geq p_j$.

### 2. Virtual Nodes (The Secret Sauce)
- Direct hashing can lead to an uneven ring (one server owning 60% of the circle).
- To fix this, we create **Virtual Nodes**. One physical server appears 100 times on the ring (e.g., `S1_1`, `S1_2`, `S1_100`).
- This averages out the distribution and ensures that if a server dies, its load is split evenly among *all* other servers, rather than just its immediate neighbor.

## 5. Real-World Example
**Amazon DynamoDB & Apache Cassandra**:
- These databases use consistent hashing to decide which of their 1,000+ nodes should store your user profile. When a node in the data center fails (which happens every day at that scale), the database uses the hash ring logic to instantly know which "Backup" node should handle the missing data, without moving any other data in the cluster.

## 6. Code Example
Implementing a basic Consistent Hashing Ring in Python:
```python
import hashlib

class ConsistentHash:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
        if nodes:
            for node in nodes:
                self.add_node(node)

    def add_node(self, node):
        for i in range(self.replicas):
            key = self._hash(f"{node}:{i}")
            self.ring[key] = node
            self.sorted_keys.append(key)
        self.sorted_keys.sort()

    def get_node(self, string_key):
        if not self.ring: return None
        key = self._hash(string_key)
        for node_key in self.sorted_keys:
            if key <= node_key:
                return self.ring[node_key]
        return self.ring[self.sorted_keys[0]] # Wrap around

    def _hash(self, val):
        return int(hashlib.md5(val.encode()).hexdigest(), 16)
```

## 7. Design Variations & Alternatives
- **Rendezvous Hashing (Highest Random Weight)**: Another way to achieve the same result. Instead of a ring, it calculates a "Weight" for every server relative to a key and picks the winner. (Used by Google).
- **Jump Consistent Hash**: A very fast algorithm if you only need a server *index* and don't need to map specific positions on a ring.

## 8. Pros and Cons
### Pros
- **Minimal Disruption**: Only 1/N data moves when the cluster changes.
- **Horizontal Scalable**: Add servers as needed with almost zero downtime.
- **Load Balancing**: Virtual nodes provide nearly perfect distribution.

### Cons
- **Complexity**: Harder to implement and debug than `mod N`.
- **Memory**: The "Ring" (especially with virtual nodes) must be stored and looked up for every request.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Cascading Fail"**: Server A dies. Its 10% load moves to Server B. If B was already at 95% capacity, B dies. Now B's + A's load moves to C... (Solution: Virtual nodes and aggressive over-provisioning).
- **Partitioning**: If some nodes think the ring is [A, B, C] and others think it's [A, B] due to a network error, data will be written to the wrong places (Split Brain).

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Massive. This is how multi-petabyte systems are built.
- **Reliability**: Improves by isolating the impact of a single node failure.

## 11. When NOT to Use This
- **Static Clusters**: If you have 3 servers and will never have 4, `hash % 3` is faster and easier.
- **Small Datasets**: If your data is small enough to fit on one machine, don't use a distributed hash ring.

## 12. Interview Perspective
- **What interviewers expect**: The ability to draw the **Circle**. Explanation of **Virtual Nodes**. Understanding why **Hash % N** is bad for scaling. 
- **Common question**: "How do you scale a cache from 10 nodes to 100 nodes without destroying your database?"

## 13. Connections to Other Topics
- **Sharding**: Consistent hashing is the logic behind "Algorithmic Sharding."
- **Load Balancing Algorithms**: This is a specialized, state-aware algorithm.
- **Microservices**: Used to route requests to specific instances of a stateful service.

## 14. Visual Aids
- **The Ring Diagram**: Showing Nodes A, B, and C as dots on a circle, with data keys finding their nearest neighbor.
- **Virtual Node Diagram**: Multiple dots of the same color spread across the circle.

## 15. Summary (TL;DR)
**Consistent Hashing maps both data and servers to a logical ring, ensuring that when the number of servers changes, only a small, predictable fraction of data needs to be relocated.**

## 16. Further Reading & Resources
- [The Original Paper: Consistent Hashing and Random Trees](https://dl.acm.org/doi/10.1145/258533.258590)
- [Tushar Roy: Consistent Hashing Video](https://www.youtube.com/watch?v=zaRkONvyGr8) - Excellent visual explanation.
- [Medium: Consistent Hashing at Discord](https://discord.com/blog/how-discord-handles-two-and-a-half-million-concurrent-users)
- [System Design Primer: Consistent Hashing](https://github.com/donnemartin/system-design-primer#consistent-hashing)
