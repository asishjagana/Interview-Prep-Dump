# Class Design

## 1. Why Should I Care About This?
- **The Building Blocks**: Classes are the atomic units of most backend systems. If your classes are poorly designed, your entire system will be brittle, no matter how good your "High-Level Architecture" is.
- **Maintainability**: Clear class design makes code readable. If you need 30 minutes to understand what a 200-line class does, you have a design problem.
- **Ease of Change**: Good class design allows you to change internal implementation details without affecting the rest of the application.

## 2. What Problem Does This Solve?
- **Tight Coupling**: Prevents Class A from knowing too much about the internals of Class B, which leads to "Spaghetti Code."
- **God Classes**: Prevents a single class from accumulating 50 different methods and 10,000 lines of code.
- **Naive Approach**: Creating classes that are just "Folders for functions." A class should represent a concept, not just a collection of code.

## 3. Core Idea (Intuition First)
Class design is like **Packaging a Product**. 
- The **Outside** (Public interface) should be simple, clear, and hard to misuse. 
- The **Inside** (Private methods/properties) can be complex, but it's hidden away. 
- You don't need to know how the gears in a watch work to tell the time; you just look at the face.

## 4. How It Actually Works
- **Cohesion**: Does everything in the class belong together? A high-cohesion class does one thing well (e.g., `EmailSender`).
- **Coupling**: How dependent is this class on others? Low coupling is better.
- **Primitive Obsession**: Replacing raw strings/integers with meaningful classes (e.g., Use an `EmailAddress` class instead of just a `String`).
- **The Law of Demeter**: "Don't talk to strangers." A class should only talk to its immediate dependencies, not the dependencies of its dependencies.

## 5. Real-World Example
**Java's `String` Class**:
- A masterpiece of class design. It's **Immutable**. Every operation (`split`, `toUpperCase`, `trim`) returns a *new* string. This makes it thread-safe and predictable. You can't "break" a string by passing it to another function.

## 6. Code Example
Improving class design via the Law of Demeter and Encapsulation:
```python
# BAD: High coupling, violates Law of Demeter
# The caller has to dig through 3 levels to get what it wants
total = user.get_wallet().get_account().get_balance()

# GOOD: Low coupling, clean delegation
# The user class provides a high-level method; internal structure is hidden
total = user.get_total_balance()

# WITHIN THE CLASS:
class User:
    def get_total_balance(self):
        # Even if 'wallet' structure changes, only this one line needs updating
        return self.__wallet.get_balance()
```

## 7. Design Variations & Alternatives
- **Immutable Classes**: Data-only objects where state cannot be changed after creation (Record types in Java/C#, Dataclasses in Python).
- **Utility Classes**: Static-only classes (like `Math`) which are just collections of functions. (Use sparingly).
- **Active Record vs Data Mapper**: Different ways to design classes that interact with a database.

## 8. Pros and Cons
### Pros
- **Unit Testing**: Small, focused classes are easy to test in isolation.
- **Readability**: Code reads like a sentence: `user.block_account()` instead of `db.update("users", {"blocked": 1}, id=u_id)`.

### Cons
- **Fragmentation**: Too many small classes can make it hard to see the "Big Picture."
- **Performance**: In extreme cases, creating thousands of objects can put pressure on the Garbage Collector.

## 9. Failure Scenarios & What Can Go Wrong
- **Feature Envy**: Class A constantly calling Class B's getters and performing logic on B's data. That logic likely belongs inside Class B!
- **Deep Inheritance**: A class hierarchy 5 levels deep. If you change a property in the grandparent, you break 20 children you didn't even know existed.
- **Inappropriate Intimacy**: Two classes that know too much about each other's private parts, making them impossible to separate.

## 10. Impact on Scalability, Reliability & Cost
- **Reliability**: Fewer bugs because state is tightly controlled.
- **Cost**: Reduces the "Ramp-up time" for new developers joining the project.

## 11. When NOT to Use This
- **Performance Critical Hot-Loops**: Object allocation overhead can be too high in extreme low-latency scenarios.
- **Data Transfer Objects (DTOs)**: These are meant to be simple data bags; don't add complex logic to them.

## 12. Interview Perspective
- **What interviewers expect**: Recognition of **Cohesion/Coupling**. Ability to explain the **Law of Demeter**. Understanding when to use a class versus when to use a simple function.
- **Common question**: "What makes a class 'clean'?"

## 13. Connections to Other Topics
- **SOLID Principles**: Specifically SRP (Single Responsibility).
- **Domain-Driven Design**: Using Value Objects vs Entities.

## 14. Visual Aids
- **Cohesion vs Coupling Chart**: High Cohesion + Low Coupling = The "Sweet Spot."
- **Demeter's Law Diagram**: Showing a caller only talking to its direct "Neighbors."

## 15. Summary (TL;DR)
**Class design is about hiding complexity behind simple facades and ensuring every class has one clear, focused mission.**

## 16. Further Reading & Resources
- [Martin Fowler: Refactoring - Catalog of Code Smells](https://refactoring.com/catalog/)
- [The Law of Demeter - Principles of Object-Oriented Design](https://www.ccs.neu.edu/home/lieber/LoD.html)
- [Clean Code - Chapter 10: Classes](file:///c:/WorkSpace/DSA+SD+Interview/Resources/Clean_Code_Notes.md) *(Note: Placeholder link for user relevance)*
- [Refactoring.Guru: Class Design Best Practices](https://refactoring.guru/design-patterns/catalog)
