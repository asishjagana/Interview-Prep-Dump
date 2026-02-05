# Distributed Consensus (Raft, Paxos)

## 1. Why Should I Care About This?
- **The Foundation of Truth**: In a distributed system, how do 3 different servers agree on what the current "Top Score" is if the network is flaky? This is the "Hardest Problem" in Computer Science.
- **High Availability**: Consensus allows a system to stay alive even if one of its nodes is hit by a meteorite. The remaining nodes "vote" and keep moving.
- **Distributed Locking**: Preventing two microservices from updating the same record at the exact same millisecond. Consensus is the engine under the hood of tools like Zookeeper, Etcd, and Consul.

## 2. What Problem Does This Solve?
- **The Split-Brain Scenario**: Preventing two parts of your system from thinking they are both the "Leader" and making conflicting changes.
- **State Machine Replication**: Ensuring that if you play 100 transactions on Server A, Server B and C see the exact same 100 transactions in the exact same order.
- **Naive Approach**: Master-Slave replication with manual failover. This is slow, error-prone, and often leads to data loss.

## 3. Core Idea (Intuition First)
Consensus is like **A Fair Election in a Parliamentary Democracy**.
- **Nodes**: The Members of Parliament (MPs).
- **The Problem**: We need to agree on one "Prime Minister" (The Leader) and one "Law" (The Data). 
- **The Rule**: You can't just declare yourself Leader. You must ask your neighbors to vote for you. If you get a **Majority** (50% + 1), you are the Leader.
- **Consistency**: Once a majority agrees on a law, it is "Committed." Even if the Leader dies tomorrow, the new leader must respect that law because a majority of the parliament still remembers it.

## 4. How It Actually Works
### 1. Paxos (The Academic Original)
- The first widely accepted consensus algorithm.
- **Reputation**: Famous for being extremely difficult to understand and even harder to implement correctly.
- **Roles**: Proposers, Acceptors, and Learners.

### 2. Raft (The "Understandable" King)
- Designed specifically to be easier to learn than Paxos.
- **States**: Every node is either a **Follower**, **Candidate**, or **Leader**.
- **The Log**: All changes are written to an append-only log.
- **Two Phases**: 
    1. **Leader Election**: If the Leader stops sending "Heartbeats," followers become candidates and start an election.
    2. **Log Replication**: The Leader receives a request, sends it to all followers, waits for a majority to say "Got it!", then officially "Commits" it.

### 3. Quorums
- If you have 5 nodes, you need 3 to agree.
- This allows for $F$ failures in a system of $2F + 1$ nodes. (e.g., 2 nodes can die in a 5-node cluster).

## 5. Real-World Example
**The "Kubernetes" Brain (Etcd)**:
1. Kubernetes stores everything (Pods, Configs, Secrets) in **Etcd**.
2. Etcd is a distributed key-value store that uses the **Raft** algorithm.
3. If you have 3 Etcd nodes and pull the power plug on 1, Kubernetes doesn't even blink. The other 2 see the majority is still there and keep working.
4. If you pull the plug on 2 nodes (Majority lost), Etcd stops accepting changes to protect your data from becoming inconsistent. It chooses "No Data" over "Wrong Data."

## 6. Code Example
A conceptual look at a Raft heartbeat check (Pseudo-code):
```python
class RaftNode:
    def __init__(self):
        self.state = "FOLLOWER"
        self.timeout = random(150, 300) # Randomized to avoid ties
        self.last_heartbeat = now()

    def check_status(self):
        if now() - self.last_heartbeat > self.timeout:
            print("Leader is dead! I am starting an election.")
            self.state = "CANDIDATE"
            self.request_votes()

    def on_receive_heartbeat(self, leader_id):
        self.last_heartbeat = now()
        self.state = "FOLLOWER"
```

## 7. Design Variations & Alternatives
- **ZAB (Zookeeper Atomic Broadcast)**: The algorithm used by Apache Zookeeper (similar to Paxos).
- **Gossip Protocols**: Used for "Eventual Consistency" (e.g., Cassandra). It's faster but doesn't guarantee strict consensus.
- **Blockchain (Proof of Work/Stake)**: Consensus for untrusted environments (Byzantine Fault Tolerance).

## 8. Pros and Cons
### Pros
- **Strong Consistency**: Every node sees the same data at the same time (linearizability).
- **Fault Tolerance**: System stays alive through high node failure rates.
- **Automatic Recovery**: No human intervention needed for leader elections.

### Cons
- **Latency**: Every write must wait for a network round-trip to a majority of nodes.
- **Throughput**: You can't write faster than a single leader can process (Consensus is not for "Massive Write" workloads).
- **Complexity**: Getting the edge cases right (e.g., handling network partitions) is notoriously difficult.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Unstable Network"**: High latency causes followers to think the leader is dead. They start an election. During the election, they can't process data. As soon as a new leader is picked, another timeout happens. The system spend 100% of its time electing leaders and 0% doing work.
- **Disk Corruption**: A majority of nodes report "Success" for a write, but then their disks fail before the data is actually flushed. (Solution: `fsync` before acknowledging).
- **Partitions**: Node A and B are separated from C, D, and E. A and B stop (they are the minority). C, D, and E elect a new leader and keep working. When the network is fixed, A and B must "Roll back" any uncommitted work and follow the new leader.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: The gold standard for metadata and distributed locks.
- **Scalability**: Poor. Adding more nodes makes the system *slower* because the leader has more people to wait for.

## 11. When NOT to Use This
- **Massive Data**: Don't store 10TB of images in a Raft-based DB. It's for small, critical metadata (configs, locks, offsets).
- **Latent Connections**: Don't run a Raft cluster across Earth and Mars. The latency will kill the throughput.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain **Raft States** (Follower, Candidate, Leader). Knowledge of **Majority/Quorum**. Understanding of the **Split-Brain** problem.
- **Common question**: "How do 3 servers ensure they all have the same data even if one crashes?" (Answer: Raft/Paxos consensus).

## 13. Connections to Other Topics
- **Service Discovery**: Consul uses Raft.
- **Configuration Management**: Etcd/Vault use Raft.
- **CAP Theorem**: Consensus prioritizes Consistency (C) and Partition Tolerance (P) over Availability (A).

## 14. Visual Aids
- **The Election Diagram**: Node A [Candidate] -> "Vote for me?" -> Node B & C [Vote].
- **The Majority Wall**: 3 nodes on one side, 2 on the other. Only the 3 can commit.

## 15. Summary (TL;DR)
**Distributed consensus algorithms like Raft and Paxos allow a cluster of machines to agree on a single source of truth in the presence of node and network failures, forming the bedrock of highly available distributed systems.**

## 16. Further Reading & Resources
- [The Raft Paper: In Search of an Understandable Consensus Algorithm](https://raft.github.io/raft.pdf)
- [Raft Visualizer (Excellent interactive demo)](http://thesecretlivesofdata.com/raft/)
- [Paxos Made Simple (Leslie Lamport)](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)
- [Designing Data-Intensive Applications (Kleppmann)](https://dataintensive.net/) - Chapter 9 is the best overview of consensus.
