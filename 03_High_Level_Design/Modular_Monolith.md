# Modular Monolith

## 1. Why Should I Care About This?
- **The "Third Way"**: Many teams jump to microservices because their monolith is a mess. But the mess is usually due to poor internal boundaries, not the "monolith" itself.
- **Refactoring Safety**: It's much easier to fix boundaries inside one codebase than to fix them after you've split them into 20 different network-separated services.
- **Operational Zen**: You get the clean architecture of microservices with the "one-click deploy" simplicity of a monolith.

## 2. What Problem Does This Solve?
- **The "Big Ball of Mud"**: Prevents the "Payments" code from directly importing and modifying the "UI" state, which makes the system impossible to change.
- **Microservice Overhead**: Solves the issue of teams being overwhelmed by Kubernetes, Service Mesh, and Distributed Tracing before their app even has 100 users.

## 3. Core Idea (Intuition First)
A Modular Monolith is like a **LEGO set**. 
- Each feature (Payments, Auth, Shipping) is a distinct block with its own studs and holes. 
- They are all connected on the same baseplate (One deployment, one DB).
- But because they are distinct blocks, you can pull one off and replace it—or move it to a different baseplate entirely—without breaking the rest of the set.

## 4. How It Actually Works
- **Physical Boundary**: Code is separated into modules (packages, namespaces, or separate projects in the same repo).
- **Logical Boundary**: Modules can only talk to each other through defined **Public APIs** (Interfaces/Services). 
- **Strict Dependency Tree**: Module A can't see the internal classes of Module B. 
- **Database Rules**: Ideally, each module "owns" its own tables. Cross-module joins are discouraged or banned in favor of Service calls.

## 5. Real-World Example
**Shopify Core**:
- Shopify has one of the largest Ruby on Rails monoliths in the world. As they grew, it became a mess. Instead of splitting into microservices, they enforced **Modular Monolith** principles using a tool they built called `packwerk`. This allows them to maintain a single codebase where 1,000+ engineers work productively by enforcing clear boundaries.

## 6. Code Example
Enforcing boundaries at the language level:
```python
# module: catalog/public.py
class CatalogService:
    """The ONLY entry point for other modules"""
    def get_product(self, product_id):
        return _internal_db_query(product_id)

# module: orders/processor.py
# GOOD: Using the public API
from catalog.public import CatalogService
cat = CatalogService()

# BAD: Accessing internals (Should be blocked by lint/build tools)
# from catalog.internals.db_connection import CatalogDB
# CatalogDB.execute("DROP TABLE products") 
```

## 7. Design Variations & Alternatives
- **Package-by-Feature vs Package-by-Layer**: Modular Monoliths *must* package by feature.
- **Shared Kernel**: A specific module for code that *must* be shared (e.g., Domain Events, DTOs).
- **Service-Oriented Monolith**: The precursor where modules are built as if they were services but run in one process.

## 8. Pros and Cons
### Pros
- **Easy Testing**: Run all tests with one command. No need for complex "integration environments."
- **Low Latency**: Calls are simple function calls, not network requests.
- **Atomic Commits**: Change the API and the Caller in a single PR.

### Cons
- **Single Point of Failure**: A crash in one module (e.g., memory leak) crashes the entire app.
- **Scaling Limits**: You can't scale the "Image Processing" module without scaling the rest of the app.
- **Language Lock-in**: Everything must be written in the same language.

## 9. Failure Scenarios & What Can Go Wrong
- **The "Circular Dependency"**: Module A depends on B, B depends on C, C depends on A. Now you can't build or test them independently.
- **Leaky Abstractions**: Passing a "Database Entity" from one module to another. Now the consumer is coupled to the first module's private DB schema.
- **Shared Database Entanglement**: Two modules writing to the same table. This is the hardest "knot" to untie if you ever want to move to microservices.

## 10. Impact on Scalability, Reliability & Cost
- **Scalability**: Can handle vertical scaling (bigger servers) extremely well.
- **Cost**: **Lowest cost** for small-to-mid scale. Minimal DevOps overhead.
- **Reliability**: Easier to maintain high consistency because you can use standard DB transactions across modules.

## 11. When NOT to Use This
- **High-Diversity Tech Needs**: If you need one part in Python (Machine Learning) and another in Go (high-concurrency), a monolith can't support both.
- **Extreme Scale**: When you literally can't find a single server big enough to run the whole app.

## 12. Interview Perspective
- **What interviewers expect**: Recognition that moving to microservices is often a "last resort." Knowledge of how to enforce boundaries (**Packwerk**, **ArchUnit**, **Java Modules**).
- **Common question**: "How do you prevent a monolith from becoming a mess?"

## 13. Connections to Other Topics
- **Domain-Driven Design (DDD)**: The philosophy used to define module boundaries.
- **Monolith vs Microservices**: The broader architectural choice.

## 14. Visual Aids
- **The Layered vs Modular Diagram**: Showing horizontal layers (Web/Service/DB) vs vertical modules (Orders/Payments/Shipping).
- **Dependency Graph**: A clean, one-way directed acyclic graph (DAG) of modules.

## 15. Summary (TL;DR)
**Modular Monolith: The organized architecture of microservices with the operational simplicity of a monolith.**

## 16. Further Reading & Resources
- [Shopify: Under the Hood of a Modular Monolith](https://www.shopify.com/engineering/enforcing-modularity-inside-shopify-core)
- [Packwerk GitHub](https://github.com/Shopify/packwerk) - The tool used by Shopify.
- [Enforcing Architectural Rules with ArchUnit (Java)](https://www.archunit.org/)
- *Modular Monolith: A Guide* (Simon Brown)
