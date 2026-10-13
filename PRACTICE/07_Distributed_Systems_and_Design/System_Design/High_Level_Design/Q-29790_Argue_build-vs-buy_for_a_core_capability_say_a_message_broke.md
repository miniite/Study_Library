---
type: "practice"
difficulty: "Expert"
---

# Argue build-vs-buy for a core capability (say, a message broker) where buying is faster but locks you to a vendor at 10x future scale — how do you frame the decision and what escape hatches do you design in?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Service Mesh Adoption](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Service_Mesh_Adoption.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between building or buying a core capability like a message broker, as an experienced backend architect, we need to balance immediate implementation speed against long-term flexibility and scalability needs. The decision hinges on assessing the current project's scale, future growth potential, and the vendor lock-in risks associated with purchasing a third-party solution.

To frame this decision effectively, we must first understand the core requirements of our system and how they might evolve over time. Here’s a step-by-step approach to consider:

Current Requirements Analysis: Evaluate the current message broker needs in terms of throughput, latency, durability, and security. For instance, if your application primarily deals with small-scale internal communications, buying might seem like an easy choice due to quick integration.

Future Scalability Assessment: Predict the system's future scale based on user growth projections or business expansion plans. If you anticipate significant increases in message volume or diversity, building a custom solution could be more advantageous, even though it requires initial investment and effort.

Vendor Lock-In Risks: Consider whether buying a third-party solution will lock your organization into an expensive long-term contract with limited flexibility to switch vendors. For example, if the vendor’s technology does not align well with future technological trends or offers poor support for custom integrations, it might pose significant risks down the line.

Custom Solution Design: If we decide to build, focus on modular design principles that allow for future expansion and integration flexibility. For instance, designing a microservices architecture where message brokers can be decoupled from other services would enable easier replacement or upgrade of messaging capabilities in the future.

Escape Hatches Planning: Regardless of the choice, plan escape hatches to ensure you retain control over critical parts of your system. If we buy a solution, this might mean keeping the source code for key components available or negotiating terms that allow for vendor substitution without significant penalty. For building from scratch, it’s crucial to document and version all custom code so that any future need for change can be managed smoothly.

In conclusion, while buying a message broker may seem attractive due to its rapid deployment, careful consideration of long-term scalability and vendor lock-in risks is essential. A well-designed escape hatch strategy should be part of the decision-making process to mitigate these risks effectively.

#### Common mistakes
include overestimating the immediate benefits of a commercial product without considering future needs or underestimating the effort required for custom development in terms of initial costs and maintenance.

**Comparison:**

```
Scenario	Build vs. Buy Decision Factors	Custom Solution Benefits	Vendor Lock-In Risks
Small scale	Quick deployment, lower upfront cost	Easier to integrate with existing tech stack	Limited control over future upgrades
Large scale	Greater flexibility for scaling and customization	Higher initial investment, longer development time	Long-term vendor lock-in risks
```

**Code:**

```python
# Example of a simple message broker interface (pseudo-code)
class MessageBroker:
    def send_message(self, topic, message):
        pass

    def receive_messages(self, topic):
        pass

# Custom implementation example (simplified)
class CustomMessageBroker(MessageBroker):
    def __init__(self, storage_backend):
        self.storage_backend = storage_backend

    def send_message(self, topic, message):
        # Implementation details
        pass

    def receive_messages(self, topic):
        # Implementation details
        pass
```

**Example input / output:**

```text
Request:
Send a message to the 'user_activity' topic with data {"userId": 12345, "action": "login", "timestamp": "2023-10-01T14:00:00Z"}.

Response:
Message successfully sent.
```

**Data:**

```json
{
  "message": {
    "topic": "user_activity",
    "data": {
      "userId": 12345,
      "action": "login",
      "timestamp": "2023-10-01T14:00:00Z"
    }
  }
}
```

**Flow:**

1. Buy
2. Build
3. Current Requirements
4. Buy or Build?
5. Vendor Lock-In Risks
6. Custom Solution Design
7. Escape Hatches Planning
8. Plan Escape Hatches forFlexibility

