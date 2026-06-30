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

## Direct Answer

A **Sequence Diagram** is a UML behavioral diagram that shows **how different objects or components interact with each other over time** to complete a specific operation or use case.

While a **Class Diagram** shows the **static structure** of a system, a **Sequence Diagram** shows the **dynamic behavior**—the order in which messages are exchanged between objects.

---

## What Does a Sequence Diagram Show?

A sequence diagram answers questions like:

- Which components participate in a workflow?
- In what order are method calls made?
- How does a request flow through the system?
- What happens during a particular use case?

Examples:

- User login
- Place an order
- Process payment
- Send notification

---

## Basic Components

### 1. Actor

The external user or system initiating the action.

```text
User
```

### 2. Lifeline

Represents an object participating in the interaction.

```text
User        OrderService       Database
 |                |                |
 |                |                |
```

### 3. Message

Communication between objects.

```text
User -> OrderService : placeOrder()
```

### 4. Activation Bar

Represents the period during which an object is executing an operation.

```text
    |
   [ ]
   [ ]
    |
```

---

## Example: User Login Flow

```text
User          AuthService         Database
 |                 |                 |
 | login()         |                 |
 |---------------> |                 |
 |                 | validateUser()  |
 |                 |---------------> |
 |                 |                 |
 |                 | user data       |
 |                 |<--------------- |
 |                 |                 |
 | token           |                 |
 |<--------------- |                 |
```

### Flow

1. User sends login request.
2. AuthService checks credentials in Database.
3. Database returns user information.
4. AuthService generates token.
5. Token is returned to User.

---

## Example: Place Order in E-Commerce

```text
Customer     OrderService    PaymentService    InventoryService
    |               |                |                |
    | Place Order   |                |                |
    |-------------> |                |                |
    |               | Pay()          |                |
    |               |--------------> |                |
    |               |                | Success        |
    |               | <------------- |                |
    |               | Reserve Item() |                |
    |               |-------------------------------> |
    |               |                                |
    |               | <----------------------------- |
    | Order Created |                                |
    | <------------ |                                |
```

This diagram clearly shows the order of interactions.

---

## Common Notations

### Synchronous Call

Caller waits for response.

```text
ServiceA -> ServiceB : process()
```

---

### Return Message

```text
ServiceB --> ServiceA : success
```

Often omitted in interviews for simplicity.

---

### Asynchronous Call

Caller does not wait.

```text
ServiceA -->> Queue : publishEvent()
```

Example:

- Kafka
- RabbitMQ
- SQS

---

### Conditional Flow (alt)

```text
alt Payment Success
    Create Order
else Payment Failed
    Return Error
end
```

---

### Loop

```text
loop For Each Item
    Validate Item
end
```

---

## Class Diagram vs Sequence Diagram

| Class Diagram        | Sequence Diagram          |
| -------------------- | ------------------------- |
| Static view          | Dynamic view              |
| Shows classes        | Shows interactions        |
| Shows relationships  | Shows message flow        |
| Focus on structure   | Focus on behavior         |
| Used in LLD modeling | Used in workflow modeling |

---

## When Used in System Design Interviews

Sequence diagrams are useful for explaining:

- Login flow
- Payment processing
- Order placement
- Notification delivery
- Distributed service interactions
- Microservice communication

Interviewers often ask for a sequence diagram after the high-level architecture because it demonstrates that you understand the runtime behavior of the system.

---

## Interview-Ready Summary

> A UML Sequence Diagram is a behavioral diagram that shows how objects or services interact over time to complete a specific use case. It represents actors, participating components, messages exchanged between them, and the order of execution. Unlike a Class Diagram, which shows the static structure of a system, a Sequence Diagram focuses on runtime behavior and request flow.

## Question 3. What is an ER diagram?

## Direct Answer

An **ER Diagram (Entity Relationship Diagram)** is a database modeling diagram used to represent:

- **Entities** (objects or tables)
- **Attributes** (properties/columns)
- **Relationships** between entities

It is primarily used during **database design** to model how data is stored and connected before creating actual database tables.

---

## Why ER Diagrams Are Used

ER diagrams help answer questions like:

- What data needs to be stored?
- What entities exist in the system?
- How are entities related?
- What are the cardinality constraints (1:1, 1:N, M:N)?

They serve as the blueprint for designing relational databases.

---

## Core Components

### 1. Entity

An entity represents a real-world object.

Examples:

- User
- Customer
- Product
- Order

```text
+-----------+
| Customer  |
+-----------+
```

In a database, entities usually become tables.

---

### 2. Attributes

Attributes describe an entity.

Example:

```text
Customer
---------
customer_id
name
email
phone
```

Here:

- `customer_id` is an attribute
- `name` is an attribute
- `email` is an attribute

In a database, attributes become columns.

---

### 3. Relationship

Relationships show how entities are connected.

Example:

```text
Customer ---- Places ---- Order
```

Meaning:

> A customer places orders.

---

## Example ER Diagram

### E-Commerce System

```text
+------------+           +-----------+
| Customer   |           |  Order    |
+------------+           +-----------+
| customerId |-----------| orderId   |
| name       |  places   | amount    |
| email      |           | status    |
+------------+           +-----------+
```

Relationship:

- One Customer can place many Orders.

---

## Cardinality (Most Important Concept)

### 1. One-to-One (1:1)

```text
User -------- Passport
```

- One user has one passport.
- One passport belongs to one user.

---

### 2. One-to-Many (1:N)

```text
Customer -------- Order
      1            N
```

- One customer can have many orders.
- Each order belongs to one customer.

Most common relationship.

---

### 3. Many-to-Many (M:N)

```text
Student -------- Course
      M            N
```

- A student can enroll in many courses.
- A course can have many students.

Implemented using a junction table:

```text
Student
Course
StudentCourse
```

---

## Example: Social Media Database

```text
User
----
user_id
name

Post
----
post_id
content
user_id

Comment
-------
comment_id
text
post_id
user_id
```

Relationships:

```text
User 1 ------ N Post

User 1 ------ N Comment

Post 1 ------ N Comment
```

Meaning:

- A user creates many posts.
- A user writes many comments.
- A post has many comments.

---

## ER Diagram vs Class Diagram

| ER Diagram                             | Class Diagram                           |
| -------------------------------------- | --------------------------------------- |
| Used for database design               | Used for object-oriented design         |
| Models entities and data relationships | Models classes and object relationships |
| Focuses on data storage                | Focuses on application structure        |
| Tables, columns, keys                  | Classes, attributes, methods            |
| No behavior/methods                    | Includes methods                        |

### Example

ER Diagram:

```text
Customer
---------
customer_id
name
email
```

Class Diagram:

```text
Customer
---------
id
name
email
---------
createOrder()
updateProfile()
```

Notice that class diagrams contain behavior (methods), while ER diagrams focus only on data.

---

## ER Diagram vs Sequence Diagram

| ER Diagram                 | Sequence Diagram    |
| -------------------------- | ------------------- |
| Data model                 | Workflow model      |
| Static                     | Dynamic             |
| Entities and relationships | Message flow        |
| Database design            | Runtime interaction |

---

## In System Design Interviews

ER diagrams are useful when discussing:

- Database schema design
- Data modeling
- Relational databases
- Entity relationships
- Normalization

Typical interview flow:

```text
Requirements
      ↓
API Design
      ↓
ER Diagram / Data Model
      ↓
Database Design
      ↓
Scaling Strategy
```

---

## Interview-Ready Summary

> An ER Diagram (Entity Relationship Diagram) is a database modeling tool used to represent entities, their attributes, and the relationships between them. It serves as the foundation for designing relational database schemas and helps define how data is stored, connected, and queried. The most important concepts in ER modeling are entities, attributes, relationships, and cardinality (1:1, 1:N, and M:N).

## Question 4. Explain SOLID principles in system design

## Direct Answer

**SOLID** is a set of five object-oriented design principles that help create software that is:

- Easy to maintain
- Easy to extend
- Loosely coupled
- Testable
- Scalable as requirements evolve

The five principles are:

| Principle | Full Form                       |
| --------- | ------------------------------- |
| S         | Single Responsibility Principle |
| O         | Open/Closed Principle           |
| L         | Liskov Substitution Principle   |
| I         | Interface Segregation Principle |
| D         | Dependency Inversion Principle  |

In Low-Level Design interviews, SOLID principles are frequently used to evaluate code quality and design flexibility.

---

# 1. Single Responsibility Principle (SRP)

### Definition

A class should have **only one reason to change**.

In other words:

> One class = One responsibility.

### Bad Design

```javascript
class UserService {
  createUser() {
    // create user
  }

  sendEmail() {
    // send email
  }

  generateReport() {
    // generate report
  }
}
```

Problems:

- User management
- Email handling
- Reporting

All mixed into one class.

---

### Better Design

```javascript
class UserService {
  createUser() {}
}

class EmailService {
  sendEmail() {}
}

class ReportService {
  generateReport() {}
}
```

Each class has a single responsibility.

### Benefits

- Easier maintenance
- Better testing
- Lower coupling

---

# 2. Open/Closed Principle (OCP)

### Definition

Software entities should be:

> Open for extension, closed for modification.

You should add new functionality without modifying existing code.

---

### Bad Design

```javascript
class PaymentProcessor {
  process(type) {
    if (type === "creditCard") {
      // process card
    } else if (type === "paypal") {
      // process paypal
    }
  }
}
```

Adding a new payment method requires changing existing code.

---

### Better Design

```javascript
class PaymentMethod {
  pay() {}
}

class CreditCardPayment extends PaymentMethod {
  pay() {}
}

class PayPalPayment extends PaymentMethod {
  pay() {}
}

class UpiPayment extends PaymentMethod {
  pay() {}
}
```

```javascript
class PaymentProcessor {
  process(paymentMethod) {
    paymentMethod.pay();
  }
}
```

New payment methods can be added without modifying `PaymentProcessor`.

### Real-World Example

- Payment gateways
- Notification channels
- Storage providers

---

# 3. Liskov Substitution Principle (LSP)

### Definition

Subclasses should be replaceable with their parent class without breaking behavior.

---

### Bad Example

```javascript
class Bird {
  fly() {}
}

class Penguin extends Bird {
  fly() {
    throw new Error("Cannot fly");
  }
}
```

Problem:

```javascript
function makeBirdFly(bird) {
  bird.fly();
}
```

Passing a Penguin breaks the expectation.

---

### Better Design

```javascript
class Bird {}

class FlyingBird extends Bird {
  fly() {}
}

class Sparrow extends FlyingBird {}

class Penguin extends Bird {}
```

Now substitution works correctly.

### Interview Insight

If a subclass violates assumptions made by the base class, LSP is broken.

---

# 4. Interface Segregation Principle (ISP)

### Definition

Clients should not depend on methods they do not use.

Instead of one large interface, create smaller specialized interfaces.

---

### Bad Design

```javascript
class Worker {
  work() {}
  eat() {}
}
```

Suppose a robot worker exists.

```javascript
class Robot extends Worker {
  eat() {
    throw new Error();
  }
}
```

Robot doesn't need `eat()`.

---

### Better Design

```javascript
class Workable {
  work() {}
}

class Eatable {
  eat() {}
}
```

```javascript
class HumanWorker extends Workable {
  work() {}
  eat() {}
}

class RobotWorker extends Workable {
  work() {}
}
```

### Benefits

- Cleaner contracts
- Less unused code
- Easier maintenance

---

# 5. Dependency Inversion Principle (DIP)

### Definition

High-level modules should not depend on low-level modules.

Both should depend on abstractions.

---

### Bad Design

```javascript
class MySQLDatabase {
  save() {}
}

class UserService {
  constructor() {
    this.db = new MySQLDatabase();
  }
}
```

Problem:

- UserService is tightly coupled to MySQL.

Switching to PostgreSQL requires code changes.

---

### Better Design

```javascript
class Database {
  save() {}
}

class MySQLDatabase extends Database {
  save() {}
}

class PostgreSQLDatabase extends Database {
  save() {}
}
```

```javascript
class UserService {
  constructor(database) {
    this.database = database;
  }

  createUser() {
    this.database.save();
  }
}
```

Usage:

```javascript
const db = new PostgreSQLDatabase();
const service = new UserService(db);
```

### Benefits

- Easier testing
- Easier swapping implementations
- Better modularity

---

## Real System Design Examples

### Notification System

Instead of:

```javascript
if (type === "email") {
}
if (type === "sms") {
}
if (type === "push") {
}
```

Use:

```javascript
NotificationChannel.send();
```

Implementations:

- EmailChannel
- SmsChannel
- PushChannel

This follows:

- OCP
- DIP

---

### Payment System

```javascript
PaymentMethod.pay();
```

Implementations:

- CreditCardPayment
- UPI
- PayPal
- Stripe

Adding a new payment provider requires no changes to existing logic.

---

## Why SOLID Matters in Interviews

Interviewers use SOLID principles to assess:

- Code maintainability
- Extensibility
- Coupling and cohesion
- OOP design skills
- Long-term scalability of code

A design that follows SOLID is usually easier to evolve when requirements change.

---

## Interview-Ready Summary

> SOLID is a set of five object-oriented design principles that improve maintainability, flexibility, and scalability of software. SRP promotes a single responsibility per class, OCP enables extension without modification, LSP ensures subclasses can replace parent classes safely, ISP encourages small focused interfaces, and DIP reduces coupling by depending on abstractions rather than concrete implementations. Together, these principles lead to cleaner and more extensible system designs.

## Question 5. How do you design a parking lot system?

# Direct Answer

A parking lot system is a classic Low-Level Design interview problem. The goal is to design a system that can:

- Park vehicles
- Remove vehicles
- Track available spots
- Calculate parking fees
- Support multiple vehicle and spot types

The key is to model the right entities and keep the design extensible.

---

# Requirements / Problem Framing

## Functional Requirements

1. Park a vehicle
2. Unpark a vehicle
3. Generate parking ticket
4. Calculate parking fee
5. Track available spots
6. Support different vehicle types:
   - Motorcycle
   - Car
   - Truck

7. Support different parking spot types:
   - Motorcycle Spot
   - Compact Spot
   - Large Spot

## Non-Functional Requirements

- Fast spot allocation (O(log n) or O(1))
- Easy to add new vehicle types
- Thread-safe in real systems
- Extensible pricing model

---

# Core Entities

```text
ParkingLot
 ├── ParkingFloor
 │      ├── ParkingSpot
 │      ├── ParkingSpot
 │      └── ParkingSpot
 │
 ├── EntranceGate
 ├── ExitGate
 └── Ticket
```

---

# Class Diagram (Conceptual)

```text
Vehicle
 ├── Car
 ├── Motorcycle
 └── Truck

ParkingSpot
 ├── CompactSpot
 ├── MotorcycleSpot
 └── LargeSpot

ParkingFloor
ParkingLot
Ticket
Payment
```

---

# Low-Level Design (JavaScript)

## Vehicle

```javascript
class Vehicle {
  constructor(number, type) {
    this.number = number;
    this.type = type;
  }
}
```

```javascript
class Car extends Vehicle {
  constructor(number) {
    super(number, "CAR");
  }
}

class Motorcycle extends Vehicle {
  constructor(number) {
    super(number, "MOTORCYCLE");
  }
}

class Truck extends Vehicle {
  constructor(number) {
    super(number, "TRUCK");
  }
}
```

---

## Parking Spot

```javascript
class ParkingSpot {
  constructor(id, type) {
    this.id = id;
    this.type = type;
    this.isOccupied = false;
    this.vehicle = null;
  }

  assignVehicle(vehicle) {
    this.vehicle = vehicle;
    this.isOccupied = true;
  }

  removeVehicle() {
    this.vehicle = null;
    this.isOccupied = false;
  }
}
```

---

## Ticket

```javascript
class Ticket {
  constructor(ticketId, vehicle, spot) {
    this.ticketId = ticketId;
    this.vehicle = vehicle;
    this.spot = spot;
    this.entryTime = Date.now();
  }
}
```

---

## Parking Floor

```javascript
class ParkingFloor {
  constructor(id) {
    this.id = id;
    this.spots = [];
  }

  addSpot(spot) {
    this.spots.push(spot);
  }

  getAvailableSpot(vehicleType) {
    return this.spots.find(
      (spot) => !spot.isOccupied && this.canFit(vehicleType, spot.type),
    );
  }

  canFit(vehicleType, spotType) {
    const mapping = {
      MOTORCYCLE: ["MOTORCYCLE", "COMPACT", "LARGE"],
      CAR: ["COMPACT", "LARGE"],
      TRUCK: ["LARGE"],
    };

    return mapping[vehicleType].includes(spotType);
  }
}
```

---

## Parking Lot

```javascript
class ParkingLot {
  constructor() {
    this.floors = [];
    this.activeTickets = new Map();
  }

  addFloor(floor) {
    this.floors.push(floor);
  }

  parkVehicle(vehicle) {
    for (const floor of this.floors) {
      const spot = floor.getAvailableSpot(vehicle.type);

      if (spot) {
        spot.assignVehicle(vehicle);

        const ticket = new Ticket(crypto.randomUUID(), vehicle, spot);

        this.activeTickets.set(ticket.ticketId, ticket);

        return ticket;
      }
    }

    throw new Error("Parking Full");
  }

  unparkVehicle(ticketId) {
    const ticket = this.activeTickets.get(ticketId);

    if (!ticket) {
      throw new Error("Invalid Ticket");
    }

    ticket.spot.removeVehicle();

    this.activeTickets.delete(ticketId);

    return ticket;
  }
}
```

---

# Fee Calculation

Use the Strategy Pattern to support multiple pricing models.

## Fee Strategy

```javascript
class FeeStrategy {
  calculate(entryTime, exitTime) {}
}
```

---

### Hourly Pricing

```javascript
class HourlyFeeStrategy extends FeeStrategy {
  calculate(entryTime, exitTime) {
    const hours = Math.ceil((exitTime - entryTime) / (1000 * 60 * 60));

    return hours * 20;
  }
}
```

This follows:

- Open/Closed Principle
- Easy to add new pricing schemes

---

# Improving Spot Search at Scale

### Naive Approach

```text
Find first available spot
```

Complexity:

```text
O(total_spots)
```

Bad for:

```text
100,000+ spots
```

---

### Better Approach

Maintain:

```javascript
availableCompactSpots;
availableLargeSpots;
availableMotorcycleSpots;
```

using:

```text
Priority Queue
TreeSet
Min Heap
```

Now allocation becomes:

```text
O(log n)
```

or

```text
O(1)
```

depending on implementation.

---

# Design Patterns Used

| Pattern   | Usage                 |
| --------- | --------------------- |
| Strategy  | Fee calculation       |
| Factory   | Vehicle creation      |
| Singleton | ParkingLot instance   |
| Observer  | Display board updates |
| State     | Spot occupancy state  |

---

# Follow-Up Questions Interviewers Ask

### How would you show available spots on display boards?

Maintain counters:

```javascript
availableCompactCount;
availableLargeCount;
availableBikeCount;
```

Update on park/unpark.

---

### How would you support reservations?

Add:

```javascript
Reservation;
```

entity.

Reserve spot before arrival.

---

### How would you support EV charging spots?

Add:

```javascript
ElectricSpot extends ParkingSpot
```

No changes to existing code (OCP).

---

### How would you support multiple parking lots?

Introduce:

```text
ParkingLotManager
    ├── Lot1
    ├── Lot2
    └── Lot3
```

---

# Interview-Ready Summary

> I would model the system around ParkingLot, ParkingFloor, ParkingSpot, Vehicle, Ticket, and Payment entities. Vehicles are assigned compatible spots, and a ticket is generated at entry. Active tickets are tracked in memory for fast lookup. Fee calculation is implemented using the Strategy pattern so pricing rules can evolve independently. For scalability, available spots are maintained in separate data structures rather than scanning all spots. The design follows SOLID principles and can easily support reservations, EV charging, display boards, and multiple parking lots.

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
