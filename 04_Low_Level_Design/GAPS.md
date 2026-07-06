# Gap Analysis — 04_Low_Level_Design

> Audit date: 2026-07-05. Existing files: 10. Missing (proposed): 9 (6 core + 3 optional).

## ✅ Currently Covered

| File | Topic |
|---|---|
| Object_Oriented_Design.md | Four pillars, encapsulation, compose-over-inherit |
| SOLID_Principles.md | The five design rules |
| Design_Patterns_Overview.md | Creational/structural/behavioral survey + pattern selection |
| Class_Design.md | Cohesion, coupling, Law of Demeter |
| Interface_Design.md | Class-level contracts, small interfaces, mockability |
| Dependency_Injection.md | Constructor injection, wiring, service-locator anti-pattern |
| Domain_Driven_Design.md | Bounded contexts, ubiquitous language, aggregates |
| State_Machines.md | Lifecycles, legal transitions, illegal states unrepresentable |
| UML_Diagrams.md | Structural notation (class diagrams, relationships) |
| Sequence_Diagrams.md | Interaction notation (time, ordering, fan-out) |

## ❌ Missing Topics (proposed files)

### Core gaps — standard interview topics with no dedicated file anywhere in the repo

1. **Creational_Design_Patterns.md** — Singleton (and why it's an anti-pattern for testability), Factory Method vs Abstract Factory, Builder, Prototype, with when-to-use decision rules. (Design_Patterns_Overview.md §3 gives a one-section survey; interviewers probe individual patterns far deeper.)
2. **Structural_Design_Patterns.md** — Adapter, Decorator, Facade, Proxy, Composite, Bridge, Flyweight; Decorator vs Proxy vs Adapter distinctions are a classic interview trap. (Only two worked snippets in the overview today.)
3. **Behavioral_Design_Patterns.md** — Strategy, Observer, Command, Template Method, Chain of Responsibility, Mediator, Visitor, Iterator; Strategy vs Template Method and Observer vs pub/sub are standard follow-ups. (Only Strategy is worked in the overview.)
4. **Concurrency_Patterns_Thread_Safety.md** — Thread-safe class design: immutability, confinement, guarded state, double-checked locking, producer-consumer, readers-writer locks, actor model at the object level. (`14_Performance_Optimization_Scalability` covers parallelism vs concurrency and optimistic/pessimistic locking at the systems level, but the OOD-level "make this class thread-safe" question — a staple of LLD rounds — has no home; cross-link the two.)
5. **Error_Handling_Exception_Design.md** — Checked vs unchecked exceptions, exception hierarchies, Result/Either types, fail-fast vs graceful degradation, error contracts as part of interface design. (Only passing mentions across the folder today.)
6. **Common_LLD_Interview_Problems.md** — A practice-problems index: parking lot, elevator, vending machine, LRU cache, Splitwise, BookMyShow, chess — each mapped to the patterns/principles files it exercises. (Parking lot and elevator appear only as analogies inside existing files; the "design a parking lot" question is the single most-asked LLD prompt.)

### Optional / nice-to-have

7. **Composition_vs_Inheritance.md** — Fragile base class, LSP-driven inheritance tests, delegation and mixins. (Heavily covered inside Object_Oriented_Design.md as "compose over inherit"; split out only if that file stays a four-pillars overview.)
8. **Designing_for_Testability.md** — Seams, test doubles (mock/stub/fake), humble object pattern, DI as the enabler. (Currently scattered as side notes across Dependency_Injection.md and Interface_Design.md.)
9. **Code_Smells_Refactoring.md** — Smell catalog (god class, feature envy, shotgun surgery), refactoring-to-patterns moves; common "improve this code" interview warm-up.

## Deliberately NOT flagged (covered in other folders)

- External API contract design (REST resources, versioning, pagination) → `02_API_Interface_Design`
- Optimistic vs pessimistic locking → `14_Performance_Optimization_Scalability/Optimistic_vs_Pessimistic_Locking.md`
- Parallelism vs concurrency (systems/runtime level) → `14_Performance_Optimization_Scalability/Parallelism_vs_Concurrency.md`
- Event-driven architecture & domain events on the wire → `03_High_Level_Design/Event_Driven_Architecture.md`, `09_Message_Queues_Event_Streaming`
- CQRS (architecture-level) → already proposed in `10_Microservices_Distributed_Systems/GAPS.md` (CQRS_Pattern.md)
- Bounded contexts as service seams → covered here (Domain_Driven_Design.md), consumed by `10_Microservices_Distributed_Systems`

## Folder housekeeping

- **Interface_Design.md (04) vs `02_API_Interface_Design/` naming collision** — both say "interface"; the 04 file is class-level contracts, 02 is network-level APIs. The files are at deliberately different altitudes, but each should open with a one-line cross-reference to the other so readers land in the right place.
- **README index will need extending** — the README's mermaid map, file index, and Decision Card are hard-coded to 10 topics; update all three when gap files land (the pattern deep-dives should hang off ③ Design Patterns in the map).
