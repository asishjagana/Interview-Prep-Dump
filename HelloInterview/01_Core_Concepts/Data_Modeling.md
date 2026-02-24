# Data Modeling: Architecting for Performance and Consistency

Choosing a database is not just about SQL vs. NoSQL. It's about understanding how the data is stored on disk, how it's retrieved, and what guarantees the system provides.

---

## 1. The Core Trade-off: ACID vs. BASE

### ACID (Relational)
Relational databases focus on strict consistency and integrity.
- **Atomicity**: All or nothing.
- **Consistency**: Data stays valid.
- **Isolation**: Concurrent transactions don't interfere.
- **Durability**: Data stays saved even after a crash.

### BASE (NoSQL)
Most NoSQL systems prioritize availability and partition tolerance.
- **Basically Available**: The system guarantees availability.
- **Soft state**: The state of the system may change over time, even without input.
- **Eventual consistency**: The system will eventually become consistent.

---

## 2. Storage Internals: B-Trees vs. LSM Trees

How a database stores data on disk determines its read/write performance.

| Type | Structure | Performance Profile | Examples |
| :--- | :--- | :--- | :--- |
| **Relational** | B+ Trees | Optimized for Reads | Postgres, MySQL |
| **NoSQL** | LSM Trees | Optimized for Writes | Cassandra, RocksDB |

### LSM (Log-Structured Merge-Tree) 
LSM Trees turn random writes into sequential writes by writing to a memory buffer (Memtable) and then flushing to immutable files (SSTables) on disk. A background process merges these files.

```mermaid
graph TD
    A[Write Request] --> B[Memtable (RAM)]
    B -- Flush --> C[SSTable L0 (Disk)]
    C -- Merge/Compact --> D[SSTable L1 (Disk)]
    style B fill:#9ff
    style D fill:#f9f
```

---

## 3. Relational Schema Design: Normalization

Normalization is the process of organizing data to reduce redundancy and improve integrity.

- **1NF**: Atomic values (no lists in a cell).
- **2NF**: 1NF + All non-key columns depend on the entire primary key.
- **3NF**: 2NF + No transitive dependencies (no non-key column depends on another non-key column).

> **Senior Tip**: In high-scale systems, we often **Denormalize**. By duplicating data (e.g., storing the `username` directly in the `posts` table), we avoid expensive `JOIN` operations across shards.

---

## 4. Specialized Databases

Don't force everything into a general-purpose DB.
- **Graph Databases** (Neo4j): For highly connected data like social networks.
- **Time-Series Databases** (InfluxDB): For metrics and logs (high write volume).
- **Vector Databases** (Pinecone): For AI/ML similarity search.

---

## 5. Consistency Models

- **Strong Consistency**: After a write, every read sees that write.
- **Eventual Consistency**: Replicas will eventually sync, but reads might be stale.
- **Read-Your-Writes**: A user always sees their own updates, but might not see others'.

```mermaid
graph LR
    A[Client] -- Write --> B[Leader]
    B -- Async Sync --> C[Follower 1]
    B -- Async Sync --> D[Follower 2]
    A -- Read --> D
    Note right of D: Might return stale data
```
