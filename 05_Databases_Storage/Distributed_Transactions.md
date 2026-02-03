# Distributed Transactions (2PC, 3PC, Saga)

## 1. Why Should I Care About This?
- **The "Broken State" Risk**: In a microservices world, a single action (e.g., "Booking a Trip") involves 3 different databases: Flights, Hotels, and Payments. If you charge the user but the flight booking fails, you have a massive legal and financial mess. Distributed transactions solve this.
- **Microservices Complexity**: Standard ACID transactions only work inside *one* database. When you split your system, you "lose" ACID and must find a newer, more complex way to maintain consistency.
- **Failover & Recovery**: Knowing how to "Undo" (Rollback) an action across 5 different services is the hallmark of a Senior Backend Engineer.

## 2. What Problem Does This Solve?
- **Inconsistent State across Services**: Prevents the "I paid but didn't get the item" or "I got the item but didn't pay" scenarios.
- **Partial Failure Recovery**: Ensuring that if a multi-step process fails at step 4, steps 1, 2, and 3 are either completed or appropriately reverted.
- **Naive Approach**: Just calling 3 APIs in a row and hoping none of them fail. This works 99% of the time, but the 1% failure leads to lost revenue and customer anger.

## 3. Core Idea (Intuition First)
Distributed transactions are like **A Group Vacation Plan**.
- **2PC (Two-Phase Commit)**: One leader asks everyone, "Are you ready to go?" If *everyone* says yes, the leader says, "Okay, buy your tickets now." If even *one* person says no or doesn't answer, nobody buys anything. (Strict but brittle).
- **Saga Pattern**: Instead of one big "Yes/No," it's a series of steps. You buy the flight. If that works, you book the hotel. If the hotel fails, you call the airline and "Cancel" the flight (Compensating Transaction).

## 4. How It Actually Works
### 1. Two-Phase Commit (2PC)
- **Phase 1 (Prepare)**: Coordinator asks all participants if they can commit. They reserve resources and say "Yes."
- **Phase 2 (Commit)**: If all said yes, Coordinator sends "Commit." Else, it sends "Rollback."
- *Problem*: If the Coordinator dies during Phase 2, everyone is stuck "Waiting" and locking their resources forever.

### 2. Three-Phase Commit (3PC)
- Adds a "Pre-Commit" phase to solve the "Waiting Forever" problem of 2PC. If the coordinator dies, participants can decide to commit or abort based on the timeout. (Rarely used due to complexity).

### 3. Saga Pattern
- A sequence of local transactions. Each service performs its transaction and publishes an event.
- **Choreography**: Each service knows what to do next by listening to events. (Decentralized).
- **Orchestration**: A central "Saga Manager" tells each service what to do. (Centralized).
- **Compensating Transactions**: If Step 3 fails, the Saga runs Step 2-Undo and Step 1-Undo.

## 5. Real-World Example
**The Uber/Lyft Booking**:
1. **Order Service** creates a pending order.
2. **Payment Service** reserves the money (Saga Step 1).
3. **Driver Service** tries to find a driver (Saga Step 2).
4. **FAIL**: No driver found.
5. **Driver Service** sends "No Driver" event.
6. **Payment Service** receives event and "Refunds/Releases" the money (Compensating Transaction).
7. **Order Service** marks order as "Cancelled."

## 6. Code Example
Implementing a simple Orchestrated Saga in Python (Pseudocode):
```python
class TripSaga:
    def execute(self, user_id):
        # Step 1: Payment
        if not payment_service.charge(user_id, 100):
            return "Payment Failed"
        
        # Step 2: Flight
        if not flight_service.book("flight_123"):
            # COMPENSATE: Refund Step 1
            payment_service.refund(user_id, 100)
            return "Flight Failed, Payment Refunded"
        
        # Step 3: Hotel
        if not hotel_service.book("hotel_456"):
            # COMPENSATE: Cancel Step 2 and Refund Step 1
            flight_service.cancel("flight_123")
            payment_service.refund(user_id, 100)
            return "Hotel Failed, Trip Cancelled"

        return "Trip Success!"
```

## 7. Design Variations & Alternatives
- **TCC (Try-Confirm-Cancel)**: Similar to 2PC but at the application layer.
- **Outbox Pattern**: Ensuring that a database update and an event publication happen atomically (Critical for Sagas).

## 8. Pros and Cons
### 2PC
- **Pros**: Strong consistency (Atomic).
- **Cons**: High latency, synchronous, blocked resources (locks), single point of failure (coordinator).

### Saga
- **Pros**: Highly scalable, asynchronous, no long-term locks.
- **Cons**: Complex logic, needs compensating transactions for everything, "Eventual Consistency" (stale data).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Ghost Refund"**: A Saga fails, but the "Compensating Transaction" (Refund) also fails or is never called, leaving the user out of pocket. (Solution: Idempotency and Retries).
- **Cyclic Dependencies**: Service A triggers Saga 1, which calls Service B, which triggers Saga 2, which calls Service A... (Crash).
- **Dirty Reads**: In a Saga, Step 1 is "Committed" immediately. Another user might see that "Flight is booked" before Step 3 (Hotel) fails and cancels it.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Sagas scale much better than 2PC because they don't hold locks.
- **Reliability**: Improves system resilience by allowing parts to fail and recover gracefully.
- **Cost**: High development cost. Writing "Undo" logic for 50 services is a lot of work.

## 11. When NOT to Use This
- **Monoliths**: Just use your database's `BEGIN TRANSACTION`.
- **Non-critical flows**: If a "Log entry" fails, don't run a Saga to undo the previous log.

## 12. Interview Perspective
- **What interviewers expect**: The ability to identify when a process spans multiple databases. Understanding of **Compensating Transactions**. 2PC's limitations (Blocking).
- **Common question**: "How do you ensure consistency when moving a balance between two microservices?" (Answer: Saga or TCC).

## 13. Connections to Other Topics
- **Event-Driven Architecture**: The backbone of Choreographed Sagas.
- **Idempotency**: Crucial for retrying failed Saga steps safely.
- **ACID**: Distributed transactions are the "Distributed version" of ACID.

## 14. Visual Aids
- **2PC Sequence Diagram**: Prepare -> Yes -> Commit.
- **Saga Flowchart**: Step 1 -> Step 2 -> Fail -> Undo 1.

## 15. Summary (TL;DR)
**Distributed transactions manage consistency across different databases. 2PC is for strict-but-slow atomic locking; Sagas are for fast-but-complex eventual consistency via compensating actions.**

## 16. Further Reading & Resources
- [Microservices.io: Saga Pattern](https://microservices.io/patterns/data/saga.html) - The best resource.
- [Microsoft: Saga distributed transactions](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- [Caitie McCaffrey: The Saga Pattern](https://www.youtube.com/watch?v=1H6tQ9yK_O4) - Legendary talk.
- [2PC vs TCC vs Saga](https://medium.com/@punit_87/distributed-transactions-2pc-3pc-and-the-saga-pattern-a-comprehensive-guide-2521c76f626a)
