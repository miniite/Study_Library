---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd version and evolve an event/message schema consumed by many independent services. What's your compatibility policy (backward/forward/full) and how do you roll out a breaking change?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To version and evolve an event/message schema in a distributed system, especially when it is consumed by multiple independent services, we need to ensure backward compatibility while allowing flexibility for future changes. The key idea here is to design our schema evolution strategy with care, ensuring minimal disruptions during the transition.

#### Triaging the Question

This question asks how to handle schema versioning and evolution in a distributed system where different services consume these events or messages. The core of the issue revolves around maintaining compatibility between services while allowing for necessary changes over time. The interviewer is probing your understanding of schema management, service communication, and potential breaking changes.

#### Assumptions
Services are written in various languages.
There could be a mix of synchronous and asynchronous interactions.
Some services might not immediately adopt new versions of the schema.
We need to handle both backward and forward compatibility during transitions.

#### First-Principles Solution

Schema Versioning Strategy:

Use a versioned namespace for each event type, e.g., V1.EventName, V2.EventName.
Maintain a changelog or version history document that tracks schema changes over time.

Backward Compatibility Policy:

Ensure new versions of the schema do not break existing consumers. This might involve adding optional fields in newer schemas to allow older services to ignore them.

Forward Compatibility Policy:

Allow new producers to add fields without breaking compatibility with existing consumers, ensuring they are optional or default to a meaningful value.

Rolling Out Breaking Changes:

Use a staged rollout approach where only a subset of services adopt the new schema initially.
Monitor the system for issues and gather feedback before fully rolling out the change.

Automated Testing and Validation:

Implement automated tests to validate both old and new schemas against generated events.
Use tools like Pact or API gateways that can enforce schema validation during communication.

#### Closing Summary

In summary, a robust approach involves maintaining versioned namespaces for schema changes, ensuring backward compatibility by adding optional fields, allowing forward compatibility through carefully designed schemas, and using a staged rollout strategy to minimize disruptions. Automated testing ensures that both old and new schemas are validated against generated events, providing an additional layer of reliability.

#### Common Mistakes/Follow-ups
Failing to Plan for Backward Compatibility: Not considering how older services will handle schema changes can lead to unexpected failures.
Ignoring Forward Compatibility: Adding fields without ensuring they do not break existing implementations can cause issues down the line.
Lack of Automated Testing: Relying solely on manual testing can be error-prone and time-consuming, leading to undiscovered issues in production.
Triangulation with Examples

Imagine we have an OrderCreated event that includes fields like orderId, customerId, and totalAmount. If we need to add a new field currencyCode for international customers, the updated schema would look something like this:

JSON
{
    "version": "V2",
    "type": "orderCreated",
    "fields": {
        "orderId": 123456,
        "customerId": 7890,
        "totalAmount": 100.00,
        "currencyCode": "USD" // New field, optional for older services
    }
}

Old services would ignore the currencyCode field, while new services could use it as needed.

Triangulation with Diagram
Yes
No
Event Producer
V1.EventName
Validate Schema?
Send Event
Update Schema to V2
Rollout V2 Schema Gradually
Self-Check

This solution covers all aspects of the question, ensuring both backward and forward compatibility while providing a structured approach to rolling out schema changes.

**Code:**

```

{ "version": "V1", "type": "orderCreated", "fields": { "orderId": 123456, "customerId": 7890, "totalAmount": 100.00 } }

CODE
Copy

```json
{
    "version": "V2",
    "type": "orderCreated",
    "fields": {
        "orderId": 123456,
        "customerId": 7890,
        "totalAmount": 100.00,
        "currencyCode": "USD" // New field
    }
}
```

