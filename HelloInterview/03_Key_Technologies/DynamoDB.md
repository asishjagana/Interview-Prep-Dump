# DynamoDB: Serverless Scaling and the Power of Keys

DynamoDB is a fully managed, multi-region NoSQL database that provides single-digit millisecond latency at any scale.

---

## 1. Data Modeling via Keys

DynamoDB's performance relies entirely on how you design your keys.
- **Partition Key (PK)**: Used to determine the physical partition. Should have high cardinality (e.g., `user_id`).
- **Sort Key (SK)**: Used to group and sort data within a partition. Enables range queries (`BETWEEN`, `BEGINS_WITH`).

**The "Inverted Index" Pattern**: Swapping PK and SK to enable reverse lookups.

---

## 2. Secondary Indexes: GSI vs. LSI

| Feature | Global Secondary Index (GSI) | Local Secondary Index (LSI) |
| :--- | :--- | :--- |
| **PK** | Can be different from base table | Must be same as base table |
| **SK** | Can be different from base table | Must be different from base table |
| **Consistency** | Eventually Consistent | Eventual or Strong |
| **Storage** | Separate physical partitions | Shared partitions |
| **Lifecycle** | Can be added/deleted anytime | Must be created at table creation |

---

## 3. Dealing with Hot Partitions

If 90% of your requests hit one `user_id`, that partition will throttle. 
- **Adaptive Capacity**: DynamoDB automatically increases throughput for hot partitions by re-allocating unused capacity from cold ones.
- **Sharding the Key**: Append a random suffix (e.g., `user:123:01`) to distribute the load.

---

## 4. Advanced Features

- **DAX (DynamoDB Accelerator)**: An in-memory cache that sits in front of DynamoDB, reducing latency from milliseconds to microseconds.
- **Streams**: A sub-second log of changes (CDC) that can trigger Lambda functions. Great for materialized views.
- **TTL (Time to Live)**: Automatically delete items after a certain timestamp without consuming Write Capacity Units (WCU).

---

## 5. Summary: When to use it?

- **Use it if**: You are on AWS, need serverless scaling, and have predictable query patterns.
- **Skip it if**: You have complex join requirements, ad-hoc aggregation needs, or need to stay vendor-neutral.

> **Senior Tip**: In an interview, always mention that DynamoDB uses **Consistent Hashing** under the hood for partition placement, even though it doesn't expose the "Ring" to users like Cassandra does.
