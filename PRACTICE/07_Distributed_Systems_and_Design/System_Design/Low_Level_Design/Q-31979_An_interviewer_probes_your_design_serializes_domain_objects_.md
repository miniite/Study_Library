---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: your design serializes domain objects directly to JSON for the API and a private field leaked to clients; how do you enforce that the wire representation is an explicit, reviewed projection rather than an accident of reflection over the domain object?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing APIs, it's critical to ensure that the data sent over the wire accurately represents what clients are allowed to see and interact with, without exposing private or unnecessary fields from your domain objects. In this scenario, you need a robust mechanism to enforce that the JSON representation of an object is explicitly defined and reviewed, rather than being an unintended byproduct of serializing the entire object state.

To address this issue, we'll adopt a low-level design approach where the explicit projection is enforced at the API boundary using DTOs (Data Transfer Objects) or projections. Here's how you can achieve this:

Define Explicit Projections: Create specific classes that map directly to the JSON representation needed for each use case.
Use Serialization Strategies: Leverage serialization frameworks like Jackson, which allow customization of what fields are included in the serialized output.
Code-First Design Approach: Ensure that your DTOs or projections are designed with a clear purpose and are explicitly reviewed.

#### Triage

The interviewer is probing for an understanding of how to enforce encapsulation at the API level without relying on accidental reflection. The goal is to ensure that only approved fields are exposed, thereby maintaining data integrity and security.

#### Assumptions
You are working in a Java environment with Jackson as the serialization library.
You have control over both the server-side domain objects and client-side code interacting with the API.

#### Solution

Define DTOs: Create Data Transfer Objects (DTOs) that represent only the fields required for external consumption. For example, if you have an Order entity with private fields like orderNumber, status, and secretCode, your DTO might look something like this:

JAVA
public class OrderDto {
    private String orderId;
    private String status;

    // Getters and setters
}

Serialization Customization: Use Jackson annotations to control the serialization process. For example, you can use @JsonIgnore to exclude certain fields or @JsonProperty to map fields differently.

JAVA
public class Order {
    private String orderId;
    @JsonIgnore
    private String status; // This field should not be exposed

    @JsonProperty("orderStatus")
    public String getStatus() {
        return status;
    }

    // Other methods and fields
}

Code-First Approach: Ensure that your DTOs are designed first, and then the domain objects can evolve independently without breaking API compatibility.

#### Closing Summary

By using explicit DTOs or projections, you enforce a clear separation between the internal state of your application and the data sent over the wire. This approach ensures that only approved fields are exposed to clients, reducing the risk of accidental exposure of sensitive information. It also promotes a cleaner design where domain objects can evolve independently of API requirements.

#### Common Mistakes
Accidental Leakage: Relying on reflection or automatic serialization without explicit control.
Overly Complicated Mapping: Overengineering mapping logic which can lead to maintenance issues and increased complexity.
Lack of Review Processes: Not having a formal review process for changes in DTOs, leading to inconsistencies.

**Code:**

```java
public class Order {
    private String orderId;
    @JsonIgnore
    private String status; // This field should not be exposed

    @JsonProperty("orderStatus")
    public String getStatus() {
        return status;
    }
}
```

