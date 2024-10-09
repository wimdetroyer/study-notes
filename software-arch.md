## Architectures

https://odrotbohm.de/2023/07/sliced-onion-architecture/
https://www.happycoders.eu/software-craftsmanship/hexagonal-architecture/

as opposed to a simple _layered_ architecture


------


There are several software architecture patterns that are used to structure software systems. These architectures offer different ways to handle complexity, scalability, testability, and maintainability, depending on the needs of the system. Here’s an overview of **major software architecture patterns**, explaining their concepts, characteristics, pros, cons, and typical use cases.

---

### **1. Layered Architecture (N-Tier Architecture)**

#### Overview:
- **Description**: A traditional architecture where the application is divided into layers, each layer having a specific role (e.g., Presentation, Business Logic, Data Access).
- **Flow**: The flow is **top-down**—the user interface (UI) interacts with the business logic, which in turn interacts with the data access layer.
  
#### Structure:
1. **Presentation Layer**: Handles the user interface and user requests.
2. **Application/Business Logic Layer**: Contains business rules, logic, and operations.
3. **Domain Layer** (sometimes merged with the Business Logic Layer): Represents core business objects (e.g., entities, domain rules).
4. **Data Access Layer**: Responsible for interacting with the database or any external data sources.

#### Pros:
- **Simplicity**: Easy to understand and implement.
- **Separation of concerns**: Each layer has a specific responsibility.
- **Modularity**: Easier to maintain individual layers.

#### Cons:
- **Tight coupling**: Layers are often tightly coupled, which reduces flexibility.
- **Limited flexibility**: Hard to isolate a specific layer for testing or reuse without dependencies.
- **Cross-cutting concerns**: Hard to handle concerns like logging, security, and caching across layers.

#### Typical Use Cases:
- **Enterprise applications**: Often used in legacy or traditional business systems, CRUD (Create, Read, Update, Delete) applications.

---

### **2. Event-Driven Architecture (EDA)**

#### Overview:
- **Description**: An architecture where communication between components happens through events. Components or services are loosely coupled and communicate asynchronously.
- **Flow**: Producers emit **events**, and consumers (subscribers) react to those events.

#### Structure:
1. **Event Producers**: Emit events when something changes in the system.
2. **Event Consumers**: Listen for specific events and react to them (e.g., update a database, trigger workflows).
3. **Event Brokers**: Middleware that distributes events to relevant consumers (e.g., message queues like Kafka, RabbitMQ).

#### Pros:
- **Scalability**: Asynchronous and loosely coupled components can scale independently.
- **Flexibility**: Easy to add new components by simply subscribing to events.
- **Resilience**: System components can fail without affecting the whole system if handled properly (retry mechanisms).

#### Cons:
- **Complex debugging**: Harder to trace and debug event flows due to the asynchronous nature.
- **Consistency**: Eventual consistency can cause temporary data inconsistencies.
- **Overhead**: Requires proper management of event queues, brokers, and processing times.

#### Typical Use Cases:
- **Distributed systems**, **microservices**, **IoT (Internet of Things)**, and applications requiring high scalability or real-time processing.

---

### **3. Microservices Architecture**

#### Overview:
- **Description**: A collection of small, independent services that communicate via APIs. Each service represents a specific business capability and can be developed, deployed, and scaled independently.
- **Flow**: Services communicate with each other either synchronously (e.g., REST APIs) or asynchronously (e.g., message queues).

#### Structure:
1. **Independent Services**: Each service has its own logic and database (optional).
2. **Service Communication**: Typically through lightweight protocols (HTTP, messaging queues).
3. **API Gateway** (optional): A gateway that routes requests from clients to the correct microservices.

#### Pros:
- **Independence**: Each service can be developed and deployed independently.
- **Scalability**: Services can scale individually, optimizing resource use.
- **Resilience**: If one service fails, others can continue to operate (with proper fault tolerance).

#### Cons:
- **Complexity**: Distributed system complexity—managing services, communication, and consistency across services.
- **Deployment overhead**: Requires DevOps practices for continuous deployment, monitoring, etc.
- **Data management**: Transactions across services can be difficult to manage.

#### Typical Use Cases:
- Large-scale applications that need to be **highly scalable** and **fault-tolerant**.
- Companies like Netflix, Amazon, and Uber use microservices.

---

### **4. Service-Oriented Architecture (SOA)**

#### Overview:
- **Description**: An older architectural style similar to microservices but typically more coarse-grained and less focused on small, independently deployable units. Services are reusable components that can interact across different platforms and technologies.
- **Flow**: Services are often exposed through an **Enterprise Service Bus (ESB)** that handles communication, orchestration, and security.

#### Structure:
1. **Services**: Self-contained units that provide a specific business function.
2. **Enterprise Service Bus (ESB)**: A middleware layer that handles service communication, routing, and orchestration.
3. **Service Registry**: A registry where services can be discovered by other services.

#### Pros:
- **Loose coupling**: Services are independent, promoting reusability and flexibility.
- **Integration**: Can integrate with legacy systems, mainframes, etc.
- **Scalability**: Individual services can be scaled as needed.

#### Cons:
- **Overhead**: Complex to set up and manage the ESB, which can become a bottleneck.
- **Latency**: Communication through the ESB can introduce latency.
- **Tightly coupled to ESB**: Changes in the ESB can affect all services.

#### Typical Use Cases:
- Large enterprise systems, where integration with legacy applications is essential.

---

### **5. Hexagonal Architecture (Ports and Adapters)**

#### Overview:
- **Description**: An architecture that isolates the core business logic (domain) from external systems (e.g., databases, UI, APIs) using **ports and adapters**. Core logic does not depend on external systems.
- **Flow**: External systems interact with the application core via **ports** (interfaces). Adapters implement those ports and translate between the external systems and the core logic.

#### Structure:
1. **Core Logic** (Domain): Contains the business rules and logic.
2. **Ports**: Interfaces that define how the core logic interacts with external systems.
3. **Adapters**: Implement the ports and handle external communications (e.g., repositories, controllers).

#### Pros:
- **Testability**: Core logic is isolated, making it easy to unit test.
- **Decoupling**: Business logic is independent of external systems, making it highly flexible.
- **Resilience**: Swapping out external systems (e.g., switching databases) can be done without impacting core logic.

#### Cons:
- **Complexity**: Requires good design discipline and understanding of dependency inversion.
- **Initial setup**: More work upfront to define ports and adapters.

#### Typical Use Cases:
- Applications that need high flexibility and long-term maintainability, like **microservices** or **enterprise-level applications**.

---

### **6. Clean Architecture**

#### Overview:
- **Description**: A layered architecture where the **core business logic** is at the center, surrounded by other layers like application logic, frameworks, and external interfaces. **Dependency inversion** is a key principle, meaning that outer layers (UI, databases, frameworks) depend on the core, not vice versa.
- **Flow**: Outer layers interact with inner layers through interfaces.

#### Structure:
1. **Entities**: Core business rules.
2. **Use Cases**: Application-specific business logic that uses the entities.
3. **Interface Adapters**: Converts data from external sources (e.g., databases, APIs, UI) to formats understood by the use cases and entities.
4. **Frameworks & Drivers**: External systems like databases, UI frameworks, etc.

#### Pros:
- **Testability**: Core business logic is isolated and easy to test.
- **Independence**: Frameworks and databases can be swapped with minimal impact on the core logic.
- **Flexibility**: Changes in business logic do not affect external systems.

#### Cons:
- **Complexity**: Requires significant initial setup and proper separation of layers.
- **Over-engineering**: Can be overkill for small or simple applications.

#### Typical Use Cases:
- **Long-living systems** that require high maintainability and testability (e.g., enterprise applications, large-scale systems).

---

### **7. Onion Architecture**

#### Overview:
- **Description**: A variant of **Clean Architecture**, but more focused on the domain model. The core domain logic is at the center, with dependencies pointing **inwards**. Outer layers (like UI, infrastructure) depend on the domain, not the other way around.
- **Flow**: External services interact with the core through adapters and interfaces, enforcing separation of concerns.

#### Structure:
1. **Core**: The domain model (entities and domain services).
2. **Service Layer**: Application-specific logic, such as use cases.
3. **Infrastructure**: Handles persistence, external APIs, UI, etc.
4. **Adapters**: Provide access to external systems through well-defined interfaces.

#### Pros:
- **Domain-focused**: Core domain logic is highly isolated and reusable.
- **Testability**: High testability due to separation of external systems from core.
- **Flexible architecture**: Makes it easier to replace external systems.

#### Cons:
- **Initial complexity**: More complex and requires disciplined design.
- **Overhead**: Extra work to define and maintain interfaces and adapters.

#### Typical Use Cases:


- Applications with **complex business rules** and logic that require long-term flexibility and testability.

---

### **8. CQRS (Command Query Responsibility Segregation)**

#### Overview:
- **Description**: An architectural pattern where **reading data** (queries) and **modifying data** (commands) are handled by separate models. Often paired with **Event Sourcing** to capture state changes as events.
- **Flow**: The **command model** handles write operations, and the **query model** handles read operations, often using different databases for each.

#### Structure:
1. **Command Model**: Handles data modifications (create, update, delete).
2. **Query Model**: Handles data reads, optimized for fast queries.
3. **Event Store** (optional): Stores state changes as events, used in conjunction with the command model.

#### Pros:
- **Performance**: Optimized for high read and write throughput.
- **Flexibility**: Different models can evolve separately for read and write operations.
- **Scalability**: Allows independent scaling of read and write operations.

#### Cons:
- **Complexity**: Requires careful synchronization between the command and query models.
- **Data consistency**: Eventual consistency between the write and read models can be an issue.

#### Typical Use Cases:
- Applications with **high scalability requirements** and **complex read-write workloads** (e.g., large-scale e-commerce systems, banking applications).

---

### **Summary Comparison Table:**

| Architecture                 | Key Characteristics                                          | Best For                                   |
|------------------------------|--------------------------------------------------------------|--------------------------------------------|
| **Layered (N-Tier)**          | Layers of responsibility, top-down flow                     | Traditional enterprise applications        |
| **Event-Driven (EDA)**        | Asynchronous, loosely coupled, event-based                  | Distributed systems, IoT, real-time apps   |
| **Microservices**             | Small, independent services, API-driven                     | Highly scalable, large-scale applications  |
| **Service-Oriented (SOA)**    | Coarse-grained services, ESB for orchestration              | Large enterprise systems                   |
| **Hexagonal**                 | Core logic isolated from external systems                   | Applications with long-term flexibility    |
| **Clean Architecture**        | Dependency inversion, core domain at center                 | Large, complex, maintainable systems       |
| **Onion Architecture**        | Domain-driven, inward-facing dependencies                   | Domain-heavy, flexible applications        |
| **CQRS**                      | Separate read and write models                              | High-throughput systems, scalable apps     |

---

### Conclusion:
The choice of architecture depends on factors such as **scalability**, **testability**, **maintainability**, and **complexity**. Traditional **Layered Architecture** is simple but rigid, whereas **Microservices**, **Event-Driven**, **Hexagonal**, and **Clean Architecture** offer more flexibility at the cost of added complexity. **CQRS** and **Onion Architecture** are especially useful when dealing with highly scalable or domain-centric systems.
