# Database Internals

Choosing the right database is half the battle in system design. This guide covers the fundamental mechanics of how databases store and replicate data.

---

## 1. SQL vs NoSQL

| Feature | SQL (Relational) | NoSQL (Non-Relational) |
| :--- | :--- | :--- |
| **Schema** | Fixed / Rigid | Dynamic / Schema-less |
| **Scaling** | Vertical (Horizontal is hard) | Horizontal (Sharding-ready) |
| **Consistency** | Strong (ACID) | Flexible (BASE) |
| **Joins** | Efficient | Hard / Denormalization required |
| **Best For** | Financial transactions, Analytics | Big data, real-time feeds, caching |

---

## 2. Database Replication

Replication is keeping a copy of the same data on multiple machines connected via a network.

### A. Leader-Follower (Master-Slave)
- **Process**: All writes go to the Leader. The Leader replicates those writes to one or more Followers. Reads can happen on any node.
- **Pro**: High read scalability. 
- **Con**: If Leader dies, a new one must be elected (Failover).

### B. Multi-Leader
- **Process**: Multiple nodes can accept writes and synchronize with each other.
- **Pro**: Better write availability and performance across geographic regions.
- **Con**: Conflict resolution is a major headache.

### C. Leaderless (Quorum-based)
- **Process**: Used in Dynamo-style DBs (Cassandra, Riak). Every node can accept reads and writes.
- **Pro**: Extremely high write availability.
- **Quorum**: `R + W > N` (Read quorum + Write quorum > Number of nodes) ensures you always see the latest write.

---

## 3. Normalization vs Denormalization

### Normalization
Reducing data redundancy by splitting data into multiple tables and using foreign keys.
- **Pro**: Data integrity, less storage used.
- **Con**: Complex JOINS slow down read performance at scale.

### Denormalization
Adding redundant data to a table to favor read performance over write performance.
- **Pro**: Ultra-fast reads (no joins).
- **Con**: Risk of data inconsistency, expensive writes (must update data in multiple places).

---

## 4. ACID: The Pillars of SQL

- **Atomicity**: "All or nothing." Either the entire transaction succeeds or fails.
- **Consistency**: The database moves from one valid state to another (constraints are maintained).
- **Isolation**: Concurrent transactions don't interfere with each other.
- **Durability**: Once a transaction is committed, it stays even if the system crashes.

---

## 5. Interview "Senior" Discussion Points

- **The "N+1" Query Problem**: How inefficient ORMs can kill database performance.
- **MVCC (Multi-Version Concurrency Control)**: How Postgres handles concurrent edits without locking everything.
- **Bloom Filters**: How NoSQL databases quickly check if a key exists on disk without doing a full scan.

> **Senior Tip**: In a distributed system, always discuss **Replication Lag**. If a user writes to the Leader and immediately reads from a Follower that hasn't synced yet, they will see "Stale Data." Use **Read-Your-Writes consistency** markers to solve this.
