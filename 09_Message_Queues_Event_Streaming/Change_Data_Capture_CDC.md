# Change Data Capture (CDC)

## 1. Why Should I Care About This?
- **The "Single Source of Truth" Problem**: You have 10 different systems (Search, Analytics, Cache, DB). If someone updates their name in the DB, how do the other 9 find out? CDC is the "Glue" that keeps them all in sync instantly.
- **Legacy Liberation**: CDC lets you turn an old, boring SQL database into a modern real-time event stream.
- **Zero-Perf Impact Syncing**: Unlike "Polling" (which slows down the DB), CDC reads from logs, meaning you can sync data without the DB even noticing.

## 2. What Problem Does This Solve?
- **Data Stale-ness**: Eliminating the 24-hour delay in "Daily Batch Loads."
- **Dual Writes Problem**: Preventing the bug where you update the DB, but the "Update Cache" call fails, leaving them out of sync.
- **Naive Approach**: Adding a "trigger" to the database or writing code in your app to `updateDB()` and `publishToKafka()` manually.

## 3. Core Idea (Intuition First)
CDC is like **A Professional Eavesdropper**.
- Every database keeps a secret diary of everything that happens (The **Transaction Log** or **Binlog**).
- Instead of asking the Database for data, the CDC tool just **reads the diary** while the database is writing to it.
- Every time it sees a "Someone added a row," it shouts that news to a message queue (Kafka). 
- It's like having a dedicated reporter inside the bank who tells you every time a deposit is made, so you don't have to call the bank manager every 5 minutes.

## 4. How It Actually Works
### 1. Log-Based CDC (The Standard)
- Tools like **Debezium** tail the DB log (MySQL binlog, Postgres WAL).
- It converts "Inserts, Updates, Deletes" into structured JSON events.
- **Benefit**: Captures 100% of changes, even those made by DB admins manually (not just your app code).

### 2. Query-Based CDC (The Backup)
- Polling the DB for a `last_updated` column.
- **Cons**: Can't detect "Deletes," high load on the DB.

### 3. Integration
- The events are typically sent to **Kafka**, where they can be consumed by Search (Elasticsearch), Caches (Redis), or Data Lakes (S3).

## 5. Real-World Example
**The "Instant Search" Experience**:
1. A customer changes their name from "Bob" to "Robert" in your app. 
2. The app writes to the **MySQL Database**. 
3. **Debezium (CDC)** notices the `UPDATE` in the MySQL Binlog. 
4. It publishes a message to Kafka: `{"op": "u", "before": "Bob", "after": "Robert"}`. 
5. **Elasticsearch Consumer** reads the message and instantly updates the search index.
6. Now, when the user searches for "Robert," they find themselves immediately. No 15-minute sync delay.

## 6. Code Example
While CDC is usually a "Deployment" of a tool, here is the structure of an event produced by Debezium:
```json
{
  "schema": { ... },
  "payload": {
    "before": { "id": 1, "name": "Bob", "v": 1 },
    "after": { "id": 1, "name": "Robert", "v": 2 },
    "source": { "version": "2.4.0", "db": "inventory", "table": "users" },
    "op": "u", 
    "ts_ms": 1706500000000
  }
}
```

## 7. Design Variations & Alternatives
- **Outbox Pattern**: The app writes the "Event" to a special `Outbox` table in the *same* DB transaction as the data. CDC then reads the Outbox table. (The most reliable way to handle business events).
- **Triggers**: DB triggers that write to a separate audit table. (Hard to maintain and slow).

## 8. Pros and Cons
### Pros
- **Real-time lower latency**.
- **Loose Coupling**: The app doesn't need to know about Elasticsearch or Redis.
- **Reliability**: Guarantees that every DB change becomes an event.

### Cons
- **Schema Evolution**: If you rename a column in SQL, your CDC event structure changes, which might break your consumers.
- **Initial Snapshot**: Loading 10 years of data from the DB into Kafka for the first time is a massive operational headache.
- **Operational Overhead**: You now have to manage Kafka Connect / Debezium clusters.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Log Rotation" Outage**: The CDC tool goes down for 2 days. The DB "Rotates" (deletes) its old logs to save space. When the CDC tool comes back, it's missing 2 days of data and can't recover. (Solution: Large log retention or full snapshots).
- **Event Bloat**: An update to a "large text" field in the DB creates a massive CDC event that exceeds Kafka's message size limit.
- **Order Confusion**: In some setups, an "Update" event might arrive at the search index *before* the "Insert" event, causing errors. (Solution: Partitioning by ID).

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves system-wide consistency.
- **Cost**: Adds the cost of Kafka storage and the "Capture" infrastructure.

## 11. When NOT to Use This
- **Small Volume**: If you have 100 users, just update the Search index in your app code.
- **In-memory Apps**: If your data doesn't live in a persistent log-based database.

## 12. Interview Perspective
- **What interviewers expect**: The realization that CDC is **Log-based**. Knowledge of **Debezium**. Understanding the **Outbox Pattern**. Ability to explain why it's better than "Dual-Writes."
- **Common question**: "How do you keep a search index in sync with a relational database with sub-second latency?" (Answer: Change Data Capture).

## 13. Connections to Other Topics
- **Exactly-once Semantics**: Crucial for ensuring CDC doesn't double-process changes.
- **Event Sourcing**: CDC is often used to "derive" state from an event stream.
- **Caching**: CDC is the "Ultimate Cache Invalidation Strategy."

## 14. Visual Aids
- **The Pipeline**: DB -> [Log Reader] -> [Kafka] -> [Consumers].
- **Comparison Diagram**: Dual-write (Multiple arrows from app) vs CDC (One arrow from app, CDC does the rest).

## 15. Summary (TL;DR)
**Change Data Capture (CDC) is a real-time data integration pattern that tracks and streams every change in a database's transaction log to external systems, ensuring high-fidelity data synchronization without impacting database performance.**

## 16. Further Reading & Resources
- [Debezium: CDC for Apache Kafka](https://debezium.io/)
- [RedHat: What is Change Data Capture?](https://www.redhat.com/en/topics/integration/what-is-change-data-capture)
- [Stripe: Engineering the Architecture of CDC](https://stripe.com/blog/change-data-capture)
- [Gunnar Morling: Debezium and the Outbox Pattern](https://debezium.io/blog/2019/02/19/reliable-microservices-data-exchange-with-the-outbox-pattern/)
