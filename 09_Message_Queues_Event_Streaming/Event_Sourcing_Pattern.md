# Event Sourcing Pattern

## 1. Why Should I Care About This?
- **Infinite Audit Log**: You don't just know the "Current Balance" of a bank account; you have a perfect, unchangeable record of every single penny that ever moved. This is a dream for compliance and debugging.
- **Time Travel**: You can recreate the entire state of your system as it existed at 2:00 PM last Tuesday. This allows you to hunt down bugs that only happened yesterday.
- **High Performance Writes**: You only ever "Append" events. There are no "Updates" or "Locks" on complex rows, which makes the system incredibly fast at accepting data.

## 2. What Problem Does This Solve?
- **Data Loss**: In traditional CRUD (Update/Delete), once you overwrite "Bob" with "Robert," the information that he was once "Bob" is gone forever. Event Sourcing saves everything.
- **Complex Projections**: Allowing different teams to build different "Views" of the same data. One team builds a "Current User Profile," while another builds a "User Activity Heatmap" using the same event stream.
- **Naive Approach**: Thinking that a database table is the "Truth." In Event Sourcing, the **Stream of Events** is the Truth; the database table is just a **Cache** of those events.

## 3. Core Idea (Intuition First)
Event Sourcing is like **An Accountant's Ledger**.
- **CRUD (Traditional)**: Like **A Blackboard**. You write "Total: $100." When someone spends $5, you erase $100 and write $95. You have no idea how you got to $95 or what happened before.
- **Event Sourcing**: Like **A Ledger Book**. You write "Initial: $100." Then "Spent: $5." Then "Found: $2." You never erase anything. To find the "Total," you just add up all the lines in the book. If you think the total is wrong, you can re-check every single line.

## 4. How It Actually Works
### 1. The Event Store
- A specialized database (like Kafka or EventStoreDB) that only allows "Appends."
- Events are **Immutable**. Once written, they can never be changed.

### 2. State Reconstruction (Replaying)
- When your app starts, it reads the stream from the beginning.
- `Apply(UserCreated)` -> `Apply(AddressUpdated)` -> `Apply(SubscriptionUpgraded)`.
- The result is the current state of that User.

### 3. Snapshots
- If you have 10 million events for a user, replaying them every time is slow.
- **The Fix**: Every 1,000 events, you save a "Snapshot" (Current state) to a separate DB. When you need the state, you load the snapshot and only replay the *new* events since then.

## 5. Real-World Example
**The "Version Control" (Git)**:
- Git is the ultimate Event Sourcing system. 
- You don't store the "Current File" only. You store a series of "Commits" (Events).
- You can `checkout` any commit and see the file exactly as it was.
- You can `merge` events from different branches.
- If the file gets corrupted, you can delete it and "Reconstruct" it by replaying the commits.

## 6. Code Example
A simple pattern for applying events to a state object in JavaScript:
```javascript
class UserAccount {
  constructor() {
    this.balance = 0;
    this.status = 'NEW';
  }

  // Pure function to apply event to state
  apply(event) {
    switch (event.type) {
      case 'ACCOUNT_OPENED':
        this.balance = event.initialDeposit;
        this.status = 'ACTIVE';
        break;
      case 'MONEY_DEPOSITED':
        this.balance += event.amount;
        break;
      case 'MONEY_WITHDRAWN':
        this.balance -= event.amount;
        break;
    }
  }
}

// Reconstructing state from an array of events
const events = [
  { type: 'ACCOUNT_OPENED', initialDeposit: 100 },
  { type: 'MONEY_DEPOSITED', amount: 50 },
  { type: 'MONEY_WITHDRAWN', amount: 20 }
];

const account = new UserAccount();
events.forEach(e => account.apply(e));
console.log(account.balance); // 130
```

## 7. Design Variations & Alternatives
- **CQRS (Command Query Responsibility Segregation)**: Often used with Event Sourcing. You have one service to "Write Events" and another service (with its own DB) to "Read State."
- **Internal Event Sourcing**: Only using events for a specific subsystem (e.g., a complex business rule) while the rest of the app uses CRUD.

## 8. Pros and Cons
### Pros
- **Atomic History**: Auditing is built-in.
- **Flexibility**: You can change your "Read DB" entirely (e.g., move from SQL to Elasticsearch) just by replaying the event stream into the new DB.
- **Concurrency**: No "Update conflicts." You just append events.

### Cons
- **Steep Learning Curve**: High "Conceptual Overhead." 
- **Eventual Consistency**: It takes a few milliseconds for the event to be processed into the "State," so a user might not see their change instantly.
- **Versioning**: If you change the format of the `UserCreated` event next year, you still have to be able to "Read" the old events from this year. (Schema Evolution is hard).

## 9. Failure Scenarios & What Can Go Wrong
- **The "Poison Event"**: An event was written with invalid data. Replaying it crashes the state builder. Since you can't delete the event (it's immutable), your system is permanently broken. (Solution: Write a **Compensating Event** that "fixes" the error).
- **Snapshot Drift**: The logic in your snapshotting code and your event-processing code gets out of sync. You load a snapshot that says "Balance: 100," but replaying the events says it should be "90."
- **N+1 Events**: You have so many events that even with snapshots, replaying is too slow for a real-time request.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Improves data safety.
- **Cost**: Increases storage costs since you keep every single change forever.

## 11. When NOT to Use This
- **Simple CRUD Apps**: If you are building a "To-Do List" app, Event Sourcing is massive over-engineering.
- **GDPR / Privacy**: It is technically difficult to "Delete" a user's data (Right to be Forgotten) in an immutable log.

## 12. Interview Perspective
- **What interviewers expect**: Ability to explain **Ledger vs Blackboard**. Knowledge of **Snapshots**. Understanding how **CQRS** fits in.
- **Common question**: "How do you handle a bug in an immutable event stream?" (Answer: You don't delete; you append a correction event).

## 13. Connections to Other Topics
- **CQRS**: The soulmate of Event Sourcing.
- **Immutable Infrastructure**: Applying the same logic to servers.
- **Distributed Transactions**: Sagas are often implemented using event sourcing.

## 14. Visual Aids
- **The Ledger Table**: Date | Event | Change | Result.
- **CQRS Flow**: UI -> Command -> Event Store -> Projector -> Read DB -> UI.

## 15. Summary (TL;DR)
**Event Sourcing is an architectural pattern where changes to the state of an application are saved as a sequence of immutable events, allowing for a perfect audit trail, time-travel debugging, and flexible data projections.**

## 16. Further Reading & Resources
- [Martin Fowler: Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Greg Young: CQRS and Event Sourcing](https://cqrs.files.wordpress.com/2010/11/cqrs_documents.pdf)
- [Microsoft: Event Sourcing Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/event-sourcing)
- [EventStoreDB: Why Event Sourcing?](https://www.eventstore.com/event-sourcing)
