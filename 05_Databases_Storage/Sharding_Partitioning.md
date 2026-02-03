# Sharding & Partitioning

## 1. Why Should I Care About This?
- **Breaking the Scaling Barrier**: A single database server has a limit. Eventually, even the biggest server in the world isn't enough for apps like YouTube or WhatsApp. Sharding is the *only* way to store and query trillions of rows.
- **Regional Performance**: Partitioning allows you to store European data in Europe and Asian data in Asia, reducing latency and complying with laws like GDPR.
- **Blast Radius Protection**: If one shard fails, only 10% of your users are affected. The other 90% stay online.

## 2. What Problem Does This Solve?
- **Vertical Scaling Limits**: Prevents the "Single Node Bottleneck" where a database becomes too slow to handle incoming writes.
- **Massive Datasets**: Solves the problem where a single server's disk space is full.
- **Naive Approach**: Buying a bigger server every 6 months. This is exponentially expensive and eventually impossible.

## 3. Core Idea (Intuition First)
- **Partitioning (Vertical/Horizontal)**: Like **Splitting an Encyclopedia into Volumes**. Instead of one 10,000-page book, you have 26 books (A, B, C...).
- **Sharding**: Like **Opening 26 Different Libraries**. Not only are the books split, but each volume is in a different building with its own staff. This allows 26 people to look up words at the exact same time without elbowing each other.

## 4. How It Actually Works
### Partitioning Types
1.  **Horizontal Partitioning (Sharding)**: Splitting a table by Rows. Users A-M go to Shard 1, N-Z go to Shard 2.
2.  **Vertical Partitioning**: Splitting a table by Columns. Put "User Profile" (frequent access) in one table and "User Bio/Settings" (rare access) in another.

### Sharding Strategies
1.  **Key-Based (Hash) Sharding**: Use a `hash(user_id) % number_of_shards` to decide where data goes. Balances load perfectly but makes adding shards difficult.
2.  **Range-Based Sharding**: Store data based on a range (e.g., Dates 2020-2021 go to Shard A). Great for time-series but creates "Hot Spots" (everybody writes to the current year shard).
3.  **Directory-Based Sharding**: A lookup table tells you which shard has which data. Flexible but the lookup table becomes a single point of failure.

## 5. Real-World Example
**Instagram's Sharding Strategy**:
- Instagram uses horizontal sharding on PostgreSQL. They use a **Custom ID Generation** logic where the ID itself contains the `Shard_ID`. This allows them to know which shard to query just by looking at the ID, without a central lookup table. Each shard is a separate Postgres instance.

## 6. Code Example
A simple Hash-based Sharding logic:
```python
import hashlib

def get_shard_id(user_handle, total_shards):
    # Convert string to a numeric hash
    hash_val = int(hashlib.md5(user_handle.encode()).hexdigest(), 16)
    # Determine shard
    return hash_val % total_shards

# Usage
shard = get_shard_id("asish_j", 10)
print(f"Connecting to Shard: {shard}") # Routing logic happens here
```

## 7. Design Variations & Alternatives
- **Consistent Hashing**: A more advanced hashing algorithm that allows you to add or remove shards with minimal data movement.
- **Read Replicas**: If your problem is "Too many reads," use replicas. If your problem is "Too much data / too many writes," use Sharding.
- **Federated Databases**: Splitting tables by function (Users DB, Orders DB) before splitting by rows.

## 8. Pros and Cons
### Pros
- **Horizontal Scalability**: Add more nodes as you grow.
- **Fault Isolation**: One shard crash doesn't mean a total outage.

### Cons
- **Joins are Impossible**: You cannot join tables across two different shards easily.
- **Operational Complexity**: Managing 100 databases is harder than managing one.
- **Rebalancing**: If Shard 1 is full but Shard 2 is empty, "moving" millions of rows while the app is live is a technical nightmare.

## 9. Failure Scenarios & What Can Go Wrong
- **Hot Spots**: One shard gets 90% of the traffic (e.g., shard containing a "Celebrity" user). The whole system lags because of one shard.
- **Resharding Pain**: Realizing your "Hash by 4" is too small and needing to "Hash by 8." This requires moving 50% of your data across the network.
- **The "Fan-out" Query**: A query that needs to ask *all* shards for an answer (e.g., "Find all users named John"). This increases latency exponentially.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Linear. 10 shards handle 10x the load of 1 shard.
- **Cost**: Increases operational cost (DevOps time) and monitoring cost.

## 11. When NOT to Use This
- **Don't Shard Prematurely**: If your database fits on one medium-sized server, sharding is a waste of time. "The first rule of sharding is: Don't shard until you must."
- **Small Datasets with High Joins**: If you need complex SQL joins, sharding will break your app.

## 12. Interview Perspective
- **What interviewers expect**: The ability to explain **Horizontal vs Vertical** partitioning. Knowledge of **Consistent Hashing**. Knowing how to handle "Celebrity Hotspots."
- **Common question**: "How would you design a database for a global chat app like WhatsApp?"

## 13. Connections to Other Topics
- **Relational vs NoSQL**: NoSQL usually comes with "Auto-sharding" built-in.
- **Consistent Hashing**: The algorithm for elite sharding.

## 14. Visual Aids
- **Sharding Diagram**: One big table being sliced into many small boxes.
- **Hash vs Range Visualization**: Showing how data distribution differs in both.

## 15. Summary (TL;DR)
**Sharding is the process of splitting a single database into multiple independent nodes based on a 'Shard Key' to achieve horizontal scale and fault isolation.**

## 16. Further Reading & Resources
- [DigitalOcean: Understanding Database Sharding](https://www.digitalocean.com/community/tutorials/understanding-database-sharding)
- [System Design Primer: Sharding](https://github.com/donnemartin/system-design-primer#sharding)
- [Instagram Engineering: Sharding at Instagram](https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a0410652)
- *Designing Data-Intensive Applications* - Chapter 6: Partitioning.
