# Batching vs Streaming

## 1. Why Should I Care About This?
- **Cost vs Speed**: Batching is cheaper (efficiency). Streaming is faster (low latency). You must decide which one your business needs.
- **System Density**: If you try to process every single event as it happens (Extreme Streaming), you might overhead your CPU with "Metadata" cost. If you wait too long (Extreme Batching), your data is stale.
- **Data Engineering**: These are the two primary ways to move data from a Database to a Warehouse (ETL).

## 2. What Problem Does This Solve?
- **Network Overhead**: Reducing the cost of "Small requests" by grouping them into "One large request."
- **Information Lag**: Moving from "Yesterday's data" to "Right now's data."
- **Naive Approach**: Doing everything as a "Single request" regardless of volume.

## 3. Core Idea (Intuition First)
Batching vs Streaming is like **The Laundry vs The Tap**:
- **Batching**: You don't wash every sock as soon as it's dirty. That would waste water and time. You wait until the basket is **Full (A Batch)**, then run the machine once. It's very efficient, but you might run out of clean socks while you wait for the basket to fill.
- **Streaming**: You turn on the tap, and the water flows **Continuously**. As soon as a drop is available, you use it. There is no "Wait," but if you only need one drop, you still have the overhead of the whole plumbing system being active.

## 4. How It Actually Works
### 1. Batch Processing (The Bucket)
- Data is collected over a period (e.g., 1 hour, or until 10,000 rows).
- A job (like **Spark** or a **Cron Job**) starts, processes all the data at once, and shuts down.
- **Key Metric**: High Throughput, High Latency.

### 2. Stream Processing (The River)
- Data is processed as soon as it is born.
- A "Long-running" worker (like **Flink** or **Kafka Streams**) stays alive 24/7.
- **Key Metric**: Low Latency, Variable Throughput.

### 3. Micro-Batching
- A hybrid approach (used by **Spark Streaming**). You process batches every 1-5 seconds. It feels like streaming but uses batch logic.

## 5. Real-World Example
**The "Bank Transaction" Statement**:
1. **Batching**: At 2:00 AM every night, the bank runs a job that calculates the interest for all 1 million customers and updates the database. This is efficient for the bank's computers.
2. **Streaming**: As soon as you swipe your card at a coffee shop, a "Fraud Detection" stream analyzes that one transaction in 50ms to decide if it should be blocked. You can't wait until 2:00 AM for fraud detection!

## 6. Code Example (Batching pattern)
Grouping updates to save Database hits:
```javascript
const batch = [];
const BATCH_SIZE = 100;

function onUserActivity(activity) {
    batch.push(activity);
    
    // 1. If the "Basket" is full, process it
    if (batch.length >= BATCH_SIZE) {
        flushBatch();
    }
}

async function flushBatch() {
    const dataToSave = [...batch];
    batch.length = 0; // Clear the basket
    
    // 2. Efficiency: 1 Query instead of 100
    await db.collection('logs').insertMany(dataToSave);
}
```

## 7. Design Variations & Alternatives
- **Lambda Architecture**: Running BOTH a Batch layer (for accuracy) and a Streaming layer (for speed) at the same time.
- **Kappa Architecture**: Treating everything as a Stream (The modern preference).
- **Windowing**: In streaming, you can still do math like "What was the average price in the LAST 5 MINUTES?" (Tumbling vs Sliding windows).

## 8. Pros and Cons
### Batching
- **Pros**: Highest efficiency (cheap). Perfect for massive "Historical" reports. Correctness is easier to guarantee.
- **Cons**: High latency. If the job fails, you might wait another day for the fix.

### Streaming
- **Pros**: Immediate results. Perfect for alerts, real-time dashboards, and fraud detection.
- **Cons**: Expensive (requires 24/7 servers). Harder to handle "Out of order" data (e.g., a message arrives late due to bad Wi-Fi).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Poison Pill" (Batch)**: The 9,999th row in a batch has a bug. It causes the whole 10,000-row job to fail. You have to find the one bad row and restart everything.
- **Backpressure (Streaming)**: Data is coming in at 1,000 eps (events per second), but your worker can only process 500 eps. The data starts "Backing up" in the queue until the queue runs out of memory and crashes. (Solution: **Autoscaling** or **Throttling**).
- **Double Counting**: A server restarts during a stream, and it processes the same message twice. (Solution: **Idempotency**).

## 10. Impact on Scalability, Reliability & Cost
- **Scaling**: Batching is easier to "Pause and Resume." Streaming requires complex auto-scaling.
- **Cost**: Batching is almost always cheaper per-record.

## 11. When NOT to Use This
- **Never Stream**: For "Tax Calculations" or "End of Month Payroll." Accuracy and batch efficiency are priority.
- **Never Batch**: For "Nuclear Reactor Temperature Monitoring." You need to know *now*.

## 12. Interview Perspective
- **What interviewers expect**: The **Laundry vs Tap** analogy. Knowledge of **RPO/RTO** implications. Understanding of **Micro-batching**.
- **Common question**: "When would you prefer Batch processing over Streaming?" (Answer: When high throughput and cost-efficiency are more important than real-time results, such as generating monthly bills or training an AI model on historical data).

## 13. Connections to Other Topics
- **Message Queues**: The "Pipe" that enables streaming.
- **MAP-REDUCE**: The classic algorithm for Batching.
- **Latency vs Throughput**: Batching = Throughput. Streaming = Latency.

## 14. Visual Aids
- **The Bucket vs The Hose**: Showing data accumulation vs continuous flow.
- **The Timeline**: Periodic spikes (Batch) vs a steady line (Streaming).

## 15. Summary (TL;DR)
**Batching groups data into large chunks to maximize processing efficiency and throughput, while Streaming processes data continuously to provide real-time insights and minimum latency; the choice depends on the business requirement for "Freshness" vs "Cost."**

## 16. Further Reading & Resources
- [Google: Dataflow - Batch and Stream processing](https://cloud.google.com/dataflow/docs/concepts/streaming-vs-batch)
- [Apache Flink: Concepts of Streaming](https://flink.apache.org/flink-architecture.html)
- [O'Reilly: Streaming Systems (Book)](https://www.oreilly.com/library/view/streaming-systems/9781491983812/)
- [Kafka vs Spark Streaming vs Flink](https://www.upsolver.com/blog/spark-streaming-vs-flink-vs-storm-vs-kafka-streams)
