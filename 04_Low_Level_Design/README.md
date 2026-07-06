# 04 · Low-Level Design 🧩

> **Where the shape becomes code.** HLD decided how many services there are and how they talk; LLD is where that decision turns into classes, contracts, and objects that a team can actually build, test, and change for years. Get the objects, boundaries, and contracts right here and a new requirement is a new class behind a stable interface; get them wrong and the cleanest architecture diagram still ships a ball of mud.

---

## 🎯 What You'll Be Able To Do

- **Model any domain as cohesive, encapsulated objects** instead of a procedural blob of shared mutable state.
- **Defend a class boundary using SOLID** — name the violation, name the fix, not just "this feels cleaner."
- **Choose and justify a design pattern** (creational, structural, behavioral) for a recurring problem, and know when *not* to reach for one.
- **Design classes and interfaces with high cohesion, low coupling, and lean contracts** that are small enough to mock and test in isolation.
- **Wire dependencies through injection** so any concrete implementation — including a test double — swaps in without touching business logic.
- **Model business rules as bounded contexts and finite state machines**, then whiteboard the result with UML and sequence diagrams so a team agrees before anyone writes code.

---

## 🗺️ How the 10 Topics Interlock

```mermaid
flowchart TD
    FOUND["Foundations — the rules objects must obey"]
    FOUND --> OOD["① Object-Oriented Design (the four pillars)"]
    FOUND --> SOLID["② SOLID Principles (five rules for change)"]

    BUILD["Building blocks — shape &amp; wire the classes"]
    BUILD --> PAT["③ Design Patterns (named recipes)"]
    BUILD --> CLS["④ Class Design (cohesion &amp; coupling)"]
    BUILD --> IFACE["⑤ Interface Design (contracts)"]
    BUILD --> DI["⑥ Dependency Injection (wiring)"]
    OOD --> BUILD
    SOLID --> BUILD

    DOMAIN["Domain &amp; behavior — model the business"]
    DOMAIN --> DDD["⑦ Domain-Driven Design (bounded contexts)"]
    DOMAIN --> SM["⑧ State Machines (lifecycles)"]
    BUILD --> DOMAIN

    COMM["Communicating the design"]
    COMM --> UML["⑨ UML Diagrams (structure notation)"]
    COMM --> SEQ["⑩ Sequence Diagrams (interaction notation)"]
    DOMAIN --> COMM

    style FOUND fill:#ede9fe,color:#000
    style BUILD fill:#cffafe,color:#000
    style DOMAIN fill:#dcfce7,color:#000
    style COMM fill:#fef3c7,color:#000
```

> **The spine:** *master the four pillars and five rules → use them to shape classes, interfaces, patterns, and the wiring between them → apply that toolkit to model the domain's language and lifecycles → draw and whiteboard the result so the team agrees before anyone commits code.* Patterns (③) are SOLID (②) made concrete; DI (⑥) is dependency inversion made runnable; UML (⑨) and sequence diagrams (⑩) are how everything above gets communicated and interview-whiteboarded.

---

## 📚 The Complete File Index

| # | Topic | One-line | Key takeaway |
|---|-------|----------|--------------|
| 1 | [Object-Oriented Design](./Object_Oriented_Design.md) | A corporation, not a spreadsheet | Encapsulate state behind guarded methods; compose over inherit; the four pillars are guarantees, not trivia |
| 2 | [SOLID Principles](./SOLID_Principles.md) | Five rules, one job each | Depend on abstractions, not concretions; one reason to change per class |
| 3 | [Design Patterns Overview](./Design_Patterns_Overview.md) | Recipes, not copy-paste code | Patterns are SOLID made concrete; isolate the seam that varies; don't force one where none fits |
| 4 | [Class Design](./Class_Design.md) | A watch: clean face, sealed gears | High cohesion + low coupling; Law of Demeter — don't talk to strangers |
| 5 | [Interface Design](./Interface_Design.md) | The wall outlet, not the appliance | Program to the contract, not the implementation; small interfaces beat fat ones; contracts are what make mocking possible |
| 6 | [Dependency Injection](./Dependency_Injection.md) | Catering, not cooking | Hand a class its tools from outside; swap real for mock by changing one wiring line |
| 7 | [Domain-Driven Design](./Domain_Driven_Design.md) | A themed amusement park, not one costume department | Split by bounded context, not a universal model; ubiquitous language kills the translator |
| 8 | [State Machines](./State_Machines.md) | A subway turnstile — one state, drawn arrows only | Make illegal states unrepresentable; every transition is a loggable, auditable event |
| 9 | [UML Diagrams](./UML_Diagrams.md) | The architect's floor plan | A filled diamond vs a hollow triangle settles ownership vs inheritance in one glyph; sketch, don't paint |
| 10 | [Sequence Diagrams](./Sequence_Diagrams.md) | A music score for a request | Time flows down; the diagram shows the order, waiting, and fan-out that code hides |

---

## 🧭 Suggested Study Paths

```mermaid
flowchart LR
    subgraph FAST["⚡ Interview Crunch"]
        F1["1 OOD"] --> F2["2 SOLID"] --> F3["3 Patterns"] --> F4["4 Class Design"] --> F5["9 UML"]
    end
    subgraph DEEP["🏗️ Mastery (full order)"]
        D1["1→2"] --> D2["3→4→5→6"] --> D3["7→8"] --> D4["9→10"]
    end
    style FAST fill:#fef3c7,color:#000
    style DEEP fill:#dcfce7,color:#000
```

| Goal | Path | Why |
|------|------|-----|
| **⚡ Interview in 48h** | 1 → 2 → 3 → 4 → 9 | Pillars + rules + patterns + class shape + how to draw it = the most-asked 80% |
| **🏗️ Deep mastery** | 1→2 → 3→4→5→6 → 7→8 → 9→10 | Rules → building blocks → domain modeling → communicating it, in dependency order |
| **🧩 "Design the classes"** | 1 → 4 → 5 → 6 | The OOD mental model, then the shape of classes/interfaces and how they're wired together |
| **🏢 Domain-modeling focus** | 2 → 7 → 8 (+ [03 · System Decomposition](../03_High_Level_Design/System_Decomposition.md)) | SOLID discipline → bounded contexts → lifecycle correctness |

> **Study each file by** attempting the 🏋️ **Practice Drills before the solutions**, rehearsing the 🎤 **Interview Q&A out loud**, and reading 🎯 **Staff+ Nuances** for senior-level depth.

---

## 🔑 The Decision Card (whole-section cheat sheet)

```
OOD       encapsulate state behind guarded methods · compose over inherit · four pillars = guarantees to neighbors, not trivia
SOLID     S=one reason to change · O=extend, don't edit · L=subtype honors the base contract · I=lean interfaces · D=depend on abstractions
PATTERNS  creational (HOW made) · structural (HOW compose) · behavioral (HOW collaborate/vary) · patterns ARE SOLID made concrete · don't force one
CLASS     high cohesion + low coupling · Law of Demeter (talk to friends, not strangers) · seal the gears, keep the face thin
INTERFACE program to the contract, not the implementation · keep it small (ISP) · this is what makes mocking possible
DI        hand in the dependency, don't build it inside · swap real→mock with one wiring-line change · avoid the Service Locator anti-pattern
DDD       ubiquitous language kills the translator · split by BOUNDED CONTEXT, not a universal model · aggregates make illegal states unrepresentable
STATE     one named state at a time · only drawn arrows are legal transitions · every transition = an auditable event
UML       shared symbol set (hollow triangle=is-a, filled diamond=owns lifecycle) · sketch to agree, don't paint for art
SEQUENCE  time flows down · shows ORDER + WAITING + FAN-OUT that code hides · design the interaction before you code it
RECURRING every layer here is the SAME idea at a different zoom: hide the how, expose a stable contract, let the pieces vary independently
```

---

## 🎨 Conventions & Authoring

- Every file follows the root [TEMPLATE.md](../TEMPLATE.md). Animated SVGs live in [/assets](../assets/).
- Each file ends with the three mastery sections: 🏋️ Practice Drills, 🎤 Interview Q&A Bank, 🎯 Staff+ Nuances.

---

## ➡️ Where This Connects

- ⬅️ [02 · API & Interface Design](../02_API_Interface_Design/) — the external contracts these internal classes and interfaces sit behind.
- ⬅️ [03 · High-Level Design](../03_High_Level_Design/) — the service boundaries and component responsibilities this section turns into code.
- ➡️ [05 · Databases & Storage](../05_Databases_Storage/) — repositories and aggregates here decide what gets persisted and how.
- ➡️ [09 · Message Queues & Event Streaming](../09_Message_Queues_Event_Streaming/) — domain events from DDD become the messages on the wire.
- ➡️ [10 · Microservices & Distributed Systems](../10_Microservices_Distributed_Systems/) — bounded contexts here are the seams services get cut along.

---
*Part of the [System Design & Architecture Master Repository](../README.md). The shape of the system was decided upstream; this is where it becomes classes and contracts that survive change.*
