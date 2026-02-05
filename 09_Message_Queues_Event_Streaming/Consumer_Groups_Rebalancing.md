# Consumer Groups & Rebalancing

## 1. Why Should I Care About This?
- **Horizontal Scale for Consumers**: If your Kafka topic is receiving 1 million messages a second, one single "Consumer" application will crash. Consumer Groups allow you to hire 100 "workers" (instances) to split the load.
- **Fault Tolerance**: If one worker dies, the group automatically detects it and hands its work over to the remaining workers.
- **Parallel Processing**: It is the primary mechanism for scaling out the "Read" side of an event-driven system.

## 2. What Problem Does This Solve?
- **Throughput Bottlenecks**: Solving the limit of single-threaded consumption.
- **Automatic Scaling**: Managing which application instance reads which data without manual configuration.
- **Naive Approach**: Having multiple app instances read the *whole* topic. This results in duplicate work (every instance sends the same email).

## 3. Core Idea (Intuition First)
Consumer Groups are like **A Team of Waiters in a Restaurant**.
- **Topic Partitions**: These are the **Tables** in the restaurant.
- **Consumer Group**: This is the **Waiter Staff**.
- **Rebalancing**: The **Manager** deciding which waiter serves which table.
    - If there are 10 tables and 2 waiters, each waiter takes 5 tables. 
    - If a 3rd waiter arrives, the manager takes some tables away from the first two and gives them to the new guy.
    - If a waiter trips and falls (crashes), the manager quickly gives his tables to the others so the customers aren't ignored.

## 4. How It Actually Works
### 1. Group Membership
- Multiple consumers join a group by sharing the same `group.id`.
- The **Group Coordinator** (a Kafka broker) tracks who is in the group.

### 2. Partition Assignment
- **Rule**: Each partition in a topic is assigned to **exactly one** consumer in a group at any time.
- If you have more consumers than partitions, some consumers will sit idle (Wasted resources).

### 3. The Rebalance
- High-priority event triggered when the group changes (node joins, node leaves, partition count changes).
- **Stop-the-world**: Traditionally, todos consumers stop reading, the assignments are recalculated, and then they start again. (Modern Kafka has incremental rebalancing to reduce this).

## 5. Real-World Example
**The "Email Notification" Service**:
- Your topic `user-events` has **12 partitions**.
- You deploy **3 instances** of your `EmailWorker`.
- **Kafka distributes**: Each instance gets **4 partitions**.
- **The Spike**: Black Friday happens. 3 workers are too slow. You deploy **3 more**.
- **Rebalance**: Kafka sees the new workers. Now each instance (6 total) gets **2 partitions**. The work is distributed evenly, and your users get their order confirmation emails twice as fast.

## 6. Code Example
Configuring a consumer group in Node.js (using `kafkajs`):
```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ clientId: 'my-app', brokers: ['localhost:9092'] });
const consumer = kafka.consumer({ groupId: 'email-notification-group' });

const run = async () => {
  await consumer.connect();
  await consumer.subscribe({ topic: 'user-events', fromBeginning: true });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      console.log({
        value: message.value.toString(),
        partition: partition, // Kafka tells you which partition this came from
      });
      // Send the actual email here...
    },
  });
};
```

## 7. Design Variations & Alternatives
- **Range vs Round Robin Assignor**: Strategies for how partitions are divided.
- **Cooperative Sticky Assignor**: The modern way to rebalance without stopping all consumption (Reducing "Rebalance Storms").
- **Fan-out**: If you have two *different* groups (`EmailGroup` and `SmsGroup`), both groups get ALL messages. Within each group, the load is split.

## 8. Pros and Cons
### Pros
- **High Throughput**: Unlimited scaling (up to the number of partitions).
- **Fault Tolerance**: Automatic failover.
- **Isolation**: Different service teams can have their own groups without interfering.

### Cons
- **Rebalance Overhead**: During a rebalance, your system might pause (Latency spikes).
- **Partition Limit**: You cannot have more workers than partitions. If you have 4 partitions, the 5th worker does nothing.
- **Stale Data**: If one consumer is slow, that specific partition falls behind while others are fast.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Rebalance Storm"**: A flaky consumer keeps joining and leaving the group every 2 seconds, causing the whole group to constantly stop and re-assign partitions. (Solution: Adjust `session.timeout.ms`).
- **Poison Message**: A message that causes the consumer to crash. The consumer dies, triggers a rebalance, the next consumer reads the same message, crashes, triggers a rebalance... (Solution: Error handling and DLQs).
- **Zombie Consumer**: A consumer that is still running but has "lost connectivity" to the coordinator. It keeps reading data and committing offsets, causing duplicates.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High, but dependent on the number of partitions defined during topic creation.
- **Reliability**: Improves significantly due to automatic failover.

## 11. When NOT to Use This
- **Small Volume**: If you have 10 messages a day, you don't need a consumer group.
- **Order Across All Events**: If you need Event 1 (User Signup) to finish before Event 10,000 (Global Report), splitting them across partitions and workers will break that ordering.

## 12. Interview Perspective
- **What interviewers expect**: Understanding of the **Relationship between partitions and consumer count**. Knowledge of **Group Coordinator**. Ability to explain **Stop-the-world rebalancing**.
- **Common question**: "What happens if you have 10 partitions and 15 consumers in a group?" (Answer: 5 consumers will be idle).

## 13. Connections to Other Topics
- **Message Queues**: The "Competing Consumer" pattern.
- **Message Ordering**: Ordering is only guaranteed per partition.
- **Horizontal Scaling**: This is the literal implementation of horizontal scale for streams.

## 14. Visual Aids
- **The Group Map**: 12 Partitions -> 3 Consumers (4 each).
- **The Failover Sequence**: Consumer 2 dies -> Coordinator notices -> Consumer 1 and 3 take over Consumer 2's partitions.

## 15. Summary (TL;DR)
**Consumer groups enable Kafka to scale horizontally by distributing partitions across multiple application instances, while the rebalancing process ensures that all partitions stay matched to an active consumer even during node failures or scaling events.**

## 16. Further Reading & Resources
- [Confluent: Incremental Cooperative Rebalancing](https://www.confluent.io/blog/cooperative-rebalancing-in-kafka-connect-community-release-0-11/)
- [Kafka Documentation: Consumer Group Management](https://kafka.apache.org/documentation/#consumerconfigs)
- [Baeldung: Kafka Consumer Groups](https://www.baeldung.com/kafka-consumer-groups)
- [Strimzi: Rebalancing Kafka Consumers](https://strimzi.io/blog/2021/01/07/rebalancing-kafka-consumers/)
