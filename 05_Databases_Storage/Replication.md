# Replication (Leader-Follower, Multi-Leader, Leaderless)

## 1. Why Should I Care About This?
- **High Availability**: If your only database server dies, your business dies. Replication ensures you have "clones" of your data ready to take over in seconds.
- **Read Scalability**: For every 1 write, a system like Twitter has 1,000 reads. Replication allows you to spread those 1,000 reads across 10-20 "Replica" servers, keeping the system fast.
- **Latency Reduction**: By placing replicas in different geographical regions (US, Europe, Asia), you can serve data to users from a server that is physically close to them.

## 2. What Problem Does This Solve?
- **Single Point of Failure (SPOF)**: Prevents data loss and downtime when a single server fails.
- **Read Bottlenecks**: Solves the issue where a single database CPU is at 100% just from processing `SELECT` queries.
- **Naive Approach**: Having a single database and hoping it never crashes. When it does, you're stuck waiting hours for a backup restore.

## 3. Core Idea (Intuition First)
Replication is like **The Newspaper Industry**.
- **Leader (Single-Leader)**: One central office (The Printing Press) writes the news. Every morning, thousands of copies (Replicas) are sent to local shops. You read the news at the shop (Replica), but the news only changes at the office (Leader).
- **Multi-Leader**: Several main offices (New York, London, Tokyo) can all write news for their region and then swap stories with each other later.
- **Leaderless**: There is no main office. Everyone just shouts news to each other, and if three people say the same thing, you believe it.

## 4. How It Actually Works
### 1. Leader-Follower (Master-Slave)
- All **Writes** go to the Leader. 
- The Leader sends a "Change Log" to Followers.
- All **Reads** can go to any Follower.
- *Problem*: If the Leader dies, you must "Elect" a new one.

### 2. Multi-Leader (Master-Master)
- Multiple nodes accept writes.
- Nodes synchronize with each other.
- *Problem*: Concurrent writes to the same row cause "Conflicts." (e.g., User A updates bio in NY, User B updates bio in London).

### 3. Leaderless (Quorum-based)
- Any node can accept a write.
- Client writes to $W$ nodes and reads from $R$ nodes.
- If $W + R > N$ (Total nodes), you are guaranteed to see the latest write. (e.g., Cassandra, Dynamo).

## 5. Real-World Example
**MySQL / PostgreSQL Read Replicas**:
- Most web apps use one "Master" Postgres DB for writes and 3-5 "Read Replicas" for the frontend. When you update your profile, it goes to the Master. When you view your profile (and everyone else's), it comes from a Replica. There is a slight "Replication Lag" (milliseconds) where you might not see your own update immediately.

## 6. Code Example
Connecting to different nodes based on operation type (Python pseudocode):
```python
class DatabaseRouter:
    def __init__(self):
        self.leader = create_conn("primary-db.internal")
        self.replicas = [create_conn("replica-1.internal"), create_conn("replica-2.internal")]

    def execute(self, query):
        if query.startswith("INSERT") or query.startswith("UPDATE"):
            # WRITES must go to Leader
            return self.leader.run(query)
        else:
            # READS can be load-balanced across replicas
            target = random.choice(self.replicas)
            return target.run(query)
```

## 7. Design Variations & Alternatives
- **Synchronous vs Asynchronous**: 
    - *Sync*: Leader waits for Follower to confirm before telling the user "Success." (Safe but slow).
    - *Async*: Leader confirms immediately. (Fast, but data might be lost if Leader dies before syncing).
- **Semi-Synchronous**: Wait for at least *one* follower to confirm.

## 8. Pros and Cons
### Leader-Follower
- **Pros**: simple, consistent reads (on leader), horizontal read scaling.
- **Cons**: Write bottleneck (one server), failover is complex (split brain).

### Multi-Leader
- **Pros**: survive regional outages, write scaling.
- **Cons**: Conflict resolution is a nightmare.

### Leaderless
- **Pros**: Extreme write availability (survive half the nodes dying).
- **Cons**: No "ordering" of events, messy reads.

## 9. Failure Scenarios & What Can Go Wrong
- **Replication Lag**: A user buys a concert ticket (Master), then refreshes (Replica), and the ticket isn't there yet. They panic and buy it again.
- **Split Brain**: After a network failure, two nodes both think they are the "Leader." Both accept writes. When the network returns, you have two different versions of the truth.
- **Follower Lag Spike**: One Follower falls 10 minutes behind. Any user hitting that node sees "the past."

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High for reads. Infinite followers = infinite read capacity.
- **Reliability**: Improves significantly. System survives node crashes.
- **Cost**: Doubles or triples your storage and compute costs ($N$ nodes = $N \times$ cost).

## 11. When NOT to Use This
- **Single-Server Prototypes**: Don't pay for 3 DBs when you have 0 users.
- **Hyper-Consistency**: If you cannot tolerate even 1ms of lag, you must read and write from the same single node.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain **Synchronous vs Asynchronous** replication. Knowledge of **Quorum ($N, W, R$)**. Understanding the **Failover** process.
- **Common question**: "How do you handle a user who just updated their profile and wants to see the change immediately in an eventually consistent system?" (Answer: Use "Read-Your-Writes" consistency or read from master for that specific user).

## 13. Connections to Other Topics
- **Consistency Models**: The result of different replication strategies.
- **High Availability**: The purpose of replication.
- **Consensus Algorithms (Paxos/Raft)**: How leaders are elected safely.

## 14. Visual Aids
- **Replication Topology**: Diagram showing Master -> Slave relationships.
- **Synchronous vs Asynchronous Timeline**: Showing the "Wait" time for the user.

## 15. Summary (TL;DR)
**Replication is the art of copying data across multiple nodes to ensure that if one fails, the system stays online, and to scale read capacity infinitely.**

## 16. Further Reading & Resources
- [MySQL Documentation: Replication Overview](https://dev.mysql.com/doc/refman/8.0/en/replication.html)
- [PostgreSQL: Streaming Replication Guide](https://wiki.postgresql.org/wiki/Streaming_Replication)
- [DigitalOcean: Master-Slave vs Master-Master Replication](https://www.digitalocean.com/community/conceptual-articles/database-replication-topologies)
- *Designing Data-Intensive Applications* - Chapter 5: Replication (The best resource on this topic).
