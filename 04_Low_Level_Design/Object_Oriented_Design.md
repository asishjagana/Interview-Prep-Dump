# Object-Oriented Design (OOD)

## 1. Why Should I Care About This?
- **Mapping Reality to Code**: OOD allows us to model complex business domains (like Bank Accounts, Users, and Vehicles) in a way that is intuitive to humans.
- **Manageable Complexity**: By breaking a system into "Objects" that own their own data and behavior, we prevent the "Everything affects everything" mess of global variables.
- **Industry Standard**: Most major enterprise systems (Java, C#, Python, C++) are built on OOD principles. To work in professional software, you must speak this language.

## 2. What Problem Does This Solve?
- **Data Corruption**: Prevents accidental modification of data by a different part of the program (Encapsulation).
- **Code Duplication**: Allows sharing common behavior (Inheritance/Composition) instead of copy-pasting the same logic across 100 pages.
- **Naive Approach**: Procedural programming for large systems. Having one giant file with 500 functions and 200 global variables. One mistake in a global variable can crash the whole system.

## 3. Core Idea (Intuition First)
OOD is about **Smart Actors**. 
- In procedural code, data is a dumb piece of paper, and the code is a person who reads and writes on it. 
- In OOD, the data *is* the person. A `BankAccount` object doesn't just hold a number; it is an actor that knows how to `deposit()`, `withdraw()`, and verify its own `balance`. You don't "change the balance"; you ask the account to perform an action.

## 4. How It Actually Works
The Four Pillars of OOD:
1.  **Encapsulation**: Hiding state and requiring all interaction through methods. (Data Protection).
2.  **Abstraction**: Hiding complexity. You know how to use a `Camera.takePhoto()` without knowing how the sensor captures light.
3.  **Inheritance**: Defining a "is-a" relationship (e.g., a `CheckingAccount` is a `BankAccount`).
4.  **Polymorphism**: The ability of different objects to respond to the same message in their own way (e.g., `Square.draw()` vs `Circle.draw()`).

## 5. Real-World Example
**The Java Collections Framework**:
- You can have a `List`. You don't care if it's an `ArrayList` (backed by a resizeable array) or a `LinkedList` (backed by nodes). Both implement the `List` interface. Your code can just call `.add()` and `.get()`. This is **Abstraction** and **Polymorphism** working perfectly at scale.

## 6. Code Example
Demonstrating Polymorphism and Encapsulation:
```python
class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount): pass

class CreditCard(PaymentMethod):
    def __init__(self, number):
        self.__number = number # Encapsulated (Private) property
        
    def pay(self, amount):
        print(f"Charging ${amount} to card ending in {self.__number[-4:]}")

class PayPal(PaymentMethod):
    def pay(self, amount):
        print(f"Processing ${amount} via PayPal API")

# The Caller doesn't care which method is used! (Polymorphism)
def checkout(payment_method: PaymentMethod, total: float):
    payment_method.pay(total)
```

## 7. Design Variations & Alternatives
- **Composition over Inheritance**: The modern preference to build complex objects by combining smaller ones instead of creating deep "grandparent -> parent -> child" hierarchies.
- **Functional Programming (FP)**: The main alternative, focusing on immutable data and pure functions.
- **Data-Oriented Design**: Used in high-performance games where memory layout and CPU cache are more important than object abstractions.

## 8. Pros and Cons
### Pros
- **Modularity**: Fix a bug in the `TaxCalculator` object, and it's fixed everywhere.
- **Reusability**: Build a `Logger` object once, use it in 50 projects.

### Cons
- **Steep Learning Curve**: Mastering OOD takes years (knowing when *not* to use inheritance is hard).
- **Over-Engineering**: It's easy to create "FactoryProviderManagerImpl" classes that add zero value but lots of lines of code.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Yo-Yo" Problem**: An inheritance chain 10 levels deep. To understand a method, you have to jump up and down the file tree.
- **Leaky Abstractions**: An object that exposes its internal data, allowing other objects to mess with its state, leading to "Impossible to reproduce" bugs.
- **God Objects**: One object that does everything (e.g., `SystemManager`), violating encapsulation and SRP.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Decoupled objects make it easier to split a system into microservices later.
- **Reliability**: Encapsulation prevents "side-effect" bugs where changing X breaks Y.
- **Cost**: Mid-to-high initial cost, but significantly lower long-term maintenance cost.

## 11. When NOT to Use This
- **Simple Scripts**: If you just need to move 5 files, don't create `FileManagerFactory` and `IOStreamInterface`.
- **Stateless Transform Pipelines**: If the goal is just `Input A -> Function B -> Output C`, functional programming is often cleaner.

## 12. Interview Perspective
- **What interviewers expect**: Clear definitions of the 4 pillars. The ability to explain "Composition vs Inheritance."
- **Common question**: "Design an Elevator System using OOD."
- **Red flag**: Not knowing why `private` variables are important or using inheritance for everything.

## 13. Connections to Other Topics
- **SOLID Principles**: The "laws" of good OOD.
- **Design Patterns**: Reusable solutions to common OOD problems.

## 14. Visual Aids
- **UML Class Diagram**: Showing properties, methods, and relationships like "has-a" and "is-a."
- **Encapsulation Visual**: A pill/capsule where data is on the inside and methods are the shell.

## 15. Summary (TL;DR)
**OOD is about organizing code into smart, independent objects that manage their own state and talk to each other through clean interfaces.**

## 16. Further Reading & Resources
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/) (The best intro to OOD thinking)
- [OOD Interview Questions for Tech Interviews](https://www.interviewbit.com/oops-interview-questions/)
- [Microsoft: Object-Oriented Programming Concepts](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/tutorials/oop)
- [MDN: Object-oriented programming (JavaScript)](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/Object-oriented_programming)
