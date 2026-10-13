---
type: "practice"
difficulty: "Expert"
---

# Design the sequencing for retiring a shared 'god database' that 25 services read and write directly. How do you introduce service ownership of data, break the direct table access, and do it without a coordinated big-bang?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ API Composition and BFF](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Idempotent_Order_APIs/API_Composition_and_BFF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of retiring a shared 'god database' that 25 services read from and write directly, we need to introduce service ownership over data while breaking the direct table access without a coordinated big-bang approach. This requires a carefully planned transition strategy involving several steps: decomposing the monolithic database into microservices, creating an event-driven architecture (EDA), and gradually migrating each service to manage its own data.

The key idea is to leverage asynchronous communication and event sourcing to enable services to evolve independently without requiring simultaneous changes across all 25 services. This approach minimizes risk and allows for a phased migration plan that can be managed over time.

**Code:**

```
MERMAID
Copy
sequenceDiagram
    participant ServiceA as Service A
    participant EventBus as EventBus
    participant ServiceB as Service B
    participant Database as Database

    ServiceA ->> Database: Read/Write Data
    Database -->> ServiceA: Return Data
    Database -->> EventBus: Emit Event
    ServiceB ->> EventBus: Subscribe to Events
    EventBus -->> ServiceB: Handle Event
```

**Flow:**

1. Monolithic God Database
2. Decompose into Microservices
3. ServiceA Reads/Writes Data
4. EventBus Publishes Events
5. ServiceB Subscribes to Events
6. Migrates Logic to HandleEvents
7. Break Direct Table AccessGradually

