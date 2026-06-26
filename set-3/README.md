# Set 3

| S.No. | Question                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is class diagram in UML?](#question-1-what-is-class-diagram-in-uml)                                                          |
| 2.    | [What is sequence diagram in UML?](#question-2-what-is-sequence-diagram-in-uml)                                                    |
| 3.    | [What is an ER diagram?](#question-3-what-is-an-er-diagram)                                                                        |
| 4.    | [Explain SOLID principles in system design](#question-4-explain-solid-principles-in-system-design)                                 |
| 5.    | [How do you design a parking lot system?](#question-5-how-do-you-design-a-parking-lot-system)                                      |
| 6.    | [How do you design a library management system?](#question-6-how-do-you-design-a-library-management-system)                        |
| 7.    | [How do you design a movie ticket booking system?](#question-7-how-do-you-design-a-movie-ticket-booking-system)                    |
| 8.    | [How do you design a hotel reservation system?](#question-8-how-do-you-design-a-hotel-reservation-system)                          |
| 9.    | [How do you design an ATM machine software?](#question-9-how-do-you-design-an-atm-machine-software)                                |
| 10.   | [How do you design a vending machine system?](#question-10-how-do-you-design-a-vending-machine-system)                             |
| 11.   | [How do you design a snake and ladder game system?](#question-11-how-do-you-design-a-snake-and-ladder-game-system)                 |
| 12.   | [How do you design a chess game system?](#question-12-how-do-you-design-a-chess-game-system)                                       |
| 13.   | [How do you design a logger utility?](#question-13-how-do-you-design-a-logger-utility)                                             |
| 14.   | [How do you design a cache implementation (like LRU cache)?](#question-14-how-do-you-design-a-cache-implementation-like-lru-cache) |
| 15.   | [How do you design a notification scheduler?](#question-15-how-do-you-design-a-notification-scheduler)                             |
| 16.   | [How do you design a social media post feed system?](#question-16-how-do-you-design-a-social-media-post-feed-system)               |
| 17.   | [How do you design a thread pool executor?](#question-17-how-do-you-design-a-thread-pool-executor)                                 |
| 18.   | [How do you design an elevator system?](#question-18-how-do-you-design-an-elevator-system)                                         |
| 19.   | [How do you design a file system?](#question-19-how-do-you-design-a-file-system)                                                   |
| 20.   | [How do you design a distributed key-value store?](#question-20-how-do-you-design-a-distributed-key-value-store)                   |

## Question 1. What is class diagram in UML?

## Direct Answer

A **Class Diagram** is a **UML (Unified Modeling Language) diagram** used to represent the **static structure of a system**. It shows:

- Classes
- Attributes (data members)
- Methods (functions)
- Relationships between classes

It is one of the most important diagrams used in **Low-Level Design (LLD)** because it models how objects and classes are organized in a software system.

---

## What a Class Diagram Represents

Think of it as a blueprint of the application's object model.

A class diagram answers questions like:

- What classes exist in the system?
- What data does each class store?
- What operations can each class perform?
- How do classes interact with each other?

---

## Basic Structure of a Class

A UML class is usually divided into three sections:

```text
+------------------+
|      User        |
+------------------+
| - id             |
| - name           |
| - email          |
+------------------+
| + login()        |
| + logout()       |
+------------------+
```

### Symbols

| Symbol | Meaning   |
| ------ | --------- |
| +      | Public    |
| -      | Private   |
| #      | Protected |

---

## Example

Consider an e-commerce system.

```text
+------------------+
|      User        |
+------------------+
| id               |
| name             |
+------------------+
| placeOrder()     |
+------------------+

          |
          | places
          |
          v

+------------------+
|      Order       |
+------------------+
| orderId          |
| amount           |
+------------------+
| calculateTotal() |
+------------------+
```

Here:

- A `User` places an `Order`
- `User` and `Order` are classes
- The line between them represents a relationship

---

## Common Relationships in Class Diagrams

### 1. Association

A general relationship between two classes.

```text
Customer -------- Order
```

Meaning:

> Customer uses or owns Orders.

---

### 2. Aggregation (Has-A Relationship)

Weak ownership.

```text
Team ◇------ Player
```

Example:

- Team has Players
- Player can exist without Team

---

### 3. Composition (Strong Has-A Relationship)

Strong ownership.

```text
House ◆------ Room
```

Example:

- Room cannot exist without House
- Deleting House deletes Rooms

---

### 4. Inheritance

Represents an **is-a** relationship.

```text
        Vehicle
           ▲
           |
   ----------------
   |              |
 Car          Bike
```

Meaning:

- Car is a Vehicle
- Bike is a Vehicle

---

### 5. Dependency

Temporary usage relationship.

```text
PaymentService -----> EmailService
```

Meaning:

- PaymentService uses EmailService
- Doesn't permanently own it

---

## Example Class Diagram (LLD)

### Online Food Delivery

```text
                 +-------------+
                 |    User     |
                 +-------------+
                 | id          |
                 | name        |
                 +-------------+
                 | placeOrder()|
                 +-------------+

                        |
                        |
                        v

                 +-------------+
                 |    Order    |
                 +-------------+
                 | orderId     |
                 | amount      |
                 +-------------+
                 | cancel()    |
                 +-------------+

                        |
                        |
                        v

                 +-------------+
                 |  Payment    |
                 +-------------+
                 | paymentId   |
                 +-------------+
                 | process()   |
                 +-------------+
```

---

## Why Class Diagrams Are Important

### In LLD Interviews

Interviewers use class diagrams to evaluate:

- Object-oriented design skills
- Class responsibilities
- Relationships between entities
- Design principles (SOLID)
- Extensibility of the system

### In Real Projects

They help:

- Communicate design among developers
- Identify dependencies early
- Reduce implementation ambiguity
- Serve as documentation

---

## Class Diagram vs Sequence Diagram

| Class Diagram             | Sequence Diagram           |
| ------------------------- | -------------------------- |
| Shows structure           | Shows behavior             |
| Static view               | Dynamic view               |
| Classes and relationships | Runtime interactions       |
| Used for object modeling  | Used for workflow modeling |

---

## Interview-Ready Summary

> A UML Class Diagram is a static design diagram that represents classes, their attributes, methods, and relationships such as association, inheritance, aggregation, and composition. It is commonly used in Low-Level Design to model the object structure of a system and communicate how different entities interact within an application.

## Question 2. What is sequence diagram in UML?

## Question 3. What is an ER diagram?

## Question 4. Explain SOLID principles in system design

## Question 5. How do you design a parking lot system?

## Question 6. How do you design a library management system?

## Question 7. How do you design a movie ticket booking system?

## Question 8. How do you design a hotel reservation system?

## Question 9. How do you design an ATM machine software?

## Question 10. How do you design a vending machine system?

## Question 11. How do you design a snake and ladder game system?

## Question 12. How do you design a chess game system?

## Question 13. How do you design a logger utility?

## Question 14. How do you design a cache implementation (like LRU cache)?

## Question 15. How do you design a notification scheduler?

## Question 16. How do you design a social media post feed system?

## Question 17. How do you design a thread pool executor?

## Question 18. How do you design an elevator system?

## Question 19. How do you design a file system?

## Question 20. How do you design a distributed key-value store?
