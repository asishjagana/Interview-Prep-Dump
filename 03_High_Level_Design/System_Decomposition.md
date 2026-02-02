# System Decomposition

## 1. Why Should I Care About This?
- **The "Tangled Web"**: Without decomposition, every part of your code depends on every other part. A change to the "User Profile" UI might brick the "Billing" backend.
- **Team Scaling**: You can't have 100 people working on one file. Decomposition is the physical process of breaking the work apart so humans can collaborate.
- **Independent Evolution**: It allows you to rewrite the "Search" system in a new language without touching the "Home Page."

## 2. What Problem Does This Solve?
- **Monolithic Entanglement**: Prevents the "Spaghetti Code" where business logic, database code, and UI are all mixed together.
- **Naive Approach**: Splitting based on technology (e.g., the "Folder approach"). Putting all CSS in one folder and all JS in another isn't decomposition; it's just organization. True decomposition is based on **Capability**.

## 3. Core Idea (Intuition First)
Decomposition is like **Organizing a Massive Library**. 
- You don't just throw all books on the floor. 
- You decompose the library into **Sections** (Genre), **Shelves** (Author), and **Individual Books**. 
- If someone wants a cookbook, they go to the Cooking section and don't disturb the people in the History section.

## 4. How It Actually Works
Two main ways to decompose:
1.  **By Subdomain (Business Capability)**: The "DDD" approach. Separate the "Ordering" logic from the "Logistics" logic. (Most recommended).
2.  **By Technical Capability**: Splitting "Real-time feeds" from "Batch reporting."
3.  **The Strangler Fig Pattern**: How to decompose an existing monolith by gradually "strangling" it with new services built on the outside.

## 5. Real-World Example
**eBay's Decomposition Journey**:
- In the early 2000s, eBay had one giant Perl monolith. As traffic exploded, they decomposed by **Functional Area**. They split into "Search," "Bidding," "Listing," and "User" services. This allowed each team to move at their own pace and use the best tools for their specific problem.

## 6. Code Example
Encapsulating a domain to prevent coupling:
```python
# Before Decomposition: Global mixed logic
def order_process(user_id, item_id):
    user = db.query(f"SELECT * FROM users WHERE id={user_id}")
    item = db.query(f"SELECT * FROM inventory WHERE id={item_id}")
    # Mixed logic
    if user['balance'] > item['price']:
        db.execute("UPDATE inventory SET stock=stock-1...")
        send_email(user['email'])

# After Decomposition: Clean Service boundaries
class OrderService:
    def process(self, user_id, item_id):
        # Service only speaks to other service interfaces, not their DBs
        user = UserService.get_profile(user_id)
        if InventoryService.is_in_stock(item_id):
            PaymentService.charge(user_id, item_id)
            NotificationService.send_confirmation(user.email)
```

## 7. Design Variations & Alternatives
- **Vertical Slicing**: Building an entire feature from UI to DB in one module (Great for fast delivery).
- **Horizontal Layering**: Splitting by Tier (Presentation, Business, Data).
- **Cell-Based Architecture**: Grouping related services into "Cells" that can be deployed as a unit (Used by AWS and Slack).

## 8. Pros and Cons
### Pros
- **Fault Isolation**: One module's failure doesn't kill the whole system.
- **Focused Testing**: You only need to test the part you changed.
- **Better Security**: You can lock down the "Vault" module while leaving the "Blog" module open.

### Cons
- **Consistency Complexity**: Joins across decomposed services are impossible (Needs Sagas/Events).
- **Inter-service Latency**: Moving from internal calls to network calls.
- **Developer Overhead**: Managing multiple repos and deployment pipelines.

## 9. Failure Scenarios & What Can Go Wrong
- **Chatty Interfaces**: Decomposing too much so that one user action triggers 50 network calls between small services (over-decomposition).
- **Misaligned Boundaries**: Putting "Price" in the Catalog service but "Discount" in the Marketing service. Now you can't show a product without talking to both.
- **The "Anemic" Service**: A service that does almost nothing but act as a proxy for a database table.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: High. Allows scaling the "Bottleneck" service without scaling the "Idle" ones.
- **Reliability**: Improves the "MTTR" (Mean Time To Repair) because the problem area is easier to find.
- **Cost**: Increases infrastructure complexity and monitoring cost.

## 11. When NOT to Use This
- **Small Teams**: The communication overhead of many services is higher than the benefit.
- **Trivial CRUD apps**: If the app just saves data and shows it, don't over-complicate the structure.

## 12. Interview Perspective
- **What interviewers expect**: Discussion on **Bounded Contexts** (DDD). How to decide where to "draw the line." Mentioning the **Strangler Pattern** for legacy apps.
- **Common question**: "How would you split a monolith into microservices?"

## 13. Connections to Other Topics
- **Domain-Driven Design (DDD)**: The logic for splitting.
- **Microservices**: The result of technical decomposition.

## 14. Visual Aids
- **The Puzzle Diagram**: Showing how separate pieces fit together to form the whole.
- **The Bounded Context Map**: Showing the overlap and boundaries of different business domains.

## 15. Summary (TL;DR)
**System Decomposition is about dividing a complex problem into smaller, manageable, and independent parts based on business capabilities.**

## 16. Further Reading & Resources
- [Martin Fowler: StranglerFigApplication](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Microservices.io: Decompose by Business Capability](https://microservices.io/patterns/decomposition/decompose-by-business-capability.html)
- [DDD Bounded Contexts Explained](https://martinfowler.com/bliki/BoundedContext.html)
- *Domain-Driven Design* (Eric Evans)
