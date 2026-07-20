# Low-Level Design (LLD)

Low-Level Design (LLD) focuses on the **detailed design of individual modules or components** of a system. It describes **how** each component is implemented, including classes, methods, relationships, data structures, algorithms, and design patterns.

Think of LLD as the **construction plan** of a building—it specifies the wiring, plumbing, and room layouts based on the architectural blueprint (HLD).

## Goal of LLD

* Define the internal implementation of each module.
* Improve code maintainability and readability.
* Promote reusability and modularity.
* Apply appropriate design patterns.
* Make development and testing easier.
* Provide a clear blueprint for developers.

## What LLD Includes

### 1. Class Design

* Classes
* Objects
* Attributes
* Methods
* Constructors

### 2. UML Class Diagrams

* Classes
* Relationships
* Multiplicity
* Visibility (`+`, `-`, `#`)

### 3. Object Relationships

* Association
* Aggregation
* Composition
* Inheritance
* Dependency

### 4. Design Patterns

* Singleton
* Factory
* Strategy
* Observer
* Builder
* Adapter
* Decorator

### 5. Data Structures & Algorithms

* Arrays
* Lists
* Maps
* Queues
* Trees
* Searching and sorting algorithms

### 6. Database Design

* Tables
* Columns
* Primary Keys
* Foreign Keys
* Constraints
* Indexes

### 7. API & Method Design

* Method signatures
* Request/Response models
* Exception handling
* Validation

### 8. Error Handling

* Custom exceptions
* Logging
* Retry logic
* Input validation

---

# Typical LLD Diagram

```text
                    +----------------+
                    |     User       |
                    +----------------+
                    | - id           |
                    | - name         |
                    +----------------+
                    | + login()      |
                    | + logout()     |
                    +----------------+
                            |
                            | Uses
                            v
                +-----------------------+
                | AuthenticationService |
                +-----------------------+
                | + authenticate()      |
                +-----------------------+
                            |
                            v
                    +----------------+
                    | UserRepository |
                    +----------------+
                    | + findById()   |
                    | + save()       |
                    +----------------+
```

---

### HLD vs LLD

| Feature  | High-Level Design (HLD)   | Low-Level Design (LLD)        |
| -------- | ------------------------- | ----------------------------- |
| Focus    | Overall architecture      | Internal component design     |
| Answers  | **What** to build         | **How** to build              |
| Level    | High                      | Detailed                      |
| Includes | Services, APIs, Databases | Classes, Methods, Interfaces  |
| Diagrams | Architecture Diagram      | Class, Sequence, UML Diagrams |
| Audience | Architects, Tech Leads    | Developers                    |
| Output   | System Blueprint          | Code Blueprint                |

---

# Real-world Examples

### 1. Parking Lot System

**Classes:**

* `ParkingLot`
* `ParkingFloor`
* `ParkingSpot`
* `Vehicle`
* `Ticket`
* `Payment`

**Design Patterns:** Singleton, Strategy

### 2. Library Management System

**Classes:**

* `Book`
* `Member`
* `Librarian`
* `BookIssue`
* `Fine`
* `Catalog`

**Concepts Used:** Inheritance, Composition

### 3. Food Delivery System

**Classes:**

* `User`
* `Restaurant`
* `Menu`
* `Order`
* `DeliveryPartner`
* `Payment`

**Design Patterns:** Factory, Strategy

### 4. ATM System

**Classes:**

* `ATM`
* `Account`
* `Card`
* `Transaction`
* `CashDispenser`

**Concepts Used:** State Pattern, Encapsulation

---

# Best Practices

* Follow **SOLID Principles**.
* Prefer **composition over inheritance**.
* Keep classes **small and focused** (Single Responsibility).
* Program to **interfaces**, not implementations.
* Use appropriate **Design Patterns** (Factory, Singleton, Strategy, Observer, Builder).
* Keep methods short and meaningful.
* Write reusable and maintainable code.
* Use proper naming conventions.
* Design for extensibility and testability.

---

# Common Mistakes

* Creating **God Classes** (one class doing everything).
* Tight coupling between classes.
* Violating SOLID principles.
* Excessive inheritance instead of composition.
* Ignoring interfaces and abstractions.
* Overusing design patterns where simple code is sufficient.
* Writing long methods with multiple responsibilities.
* Poor naming of classes and methods.
* Ignoring edge cases and error handling.

---

# Key Takeaways

1. **LLD focuses on implementation**—it defines classes, objects, methods, and their interactions.
2. **Use Object-Oriented Design principles** (SOLID, encapsulation, abstraction, inheritance, polymorphism) to build maintainable software.
3. **Design Patterns** solve recurring design problems and improve code flexibility and reusability.
4. **Loose coupling and high cohesion** make systems easier to extend, test, and maintain.
5. A good LLD should be **readable, scalable, reusable, and easy to modify** without impacting unrelated components.
