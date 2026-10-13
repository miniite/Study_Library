---
type: "practice"
difficulty: "Deep"
---

# Two senior architects disagree: event-driven choreography versus a central orchestrator for the same cross-service order flow. As the deciding principal, how do you frame the trade-offs, what data do you gather, and how do you commit without demoralizing the losing side?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve this disagreement between the architects regarding event-driven choreography versus a central orchestrator for a cross-service order flow, we need to carefully frame the trade-offs and make an informed decision based on comprehensive analysis and clear communication.

#### Triaging the Question

The key points of contention are:

Event-Driven Choreography: Services communicate asynchronously through events, with each service reacting independently to specific event types.
Central Orchestrator: A single central service coordinates all interactions, ensuring a unified flow but potentially introducing complexity and single points of failure.

#### Assumptions
The order flow involves multiple services interacting in a complex manner.
Performance, scalability, and maintainability are critical factors.
The solution needs to handle failures gracefully without compromising the integrity of transactions.

#### First-Principles Solution
Performance: Event-driven choreography can scale better since it leverages asynchronous communication and can distribute load more effectively. However, a central orchestrator ensures predictable behavior and can handle complex workflows more efficiently.
Scalability: Event-driven systems are inherently scalable as they don't rely on centralized coordination, making them suitable for microservices architectures where services might be added or removed dynamically.
Fault Tolerance: Central orchestrators offer better control over the flow but require robust error handling and retries mechanisms. Event-driven systems can handle failures more gracefully due to their event-driven nature.
Complexity: Central orchestrators introduce complexity in terms of design, implementation, and maintenance. Event-driven systems are simpler to implement but might suffer from coordination issues if not managed properly.
Gathering Data

To make an informed decision, we should gather the following data:

Service Interactions: Detailed understanding of how services interact (e.g., via REST, websockets, messaging queues).
Transaction Volume and Latency Requirements: Expected number of transactions per second and acceptable latency.
Error Handling Strategy: Current error handling mechanisms and their effectiveness in both architectures.
Future Scalability Needs: Anticipated growth in the number of services and transaction volume.
Making a Decision

Based on the gathered data, we can frame the decision as follows:

If the service interactions are relatively simple and the expected transaction volume is high, an event-driven approach might be more suitable.
For complex workflows with strict transactional requirements, a central orchestrator could provide better control and predictability.
Committing Without Demoralizing

To commit without demoralizing the losing side:

Empathize: Acknowledge both perspectives and understand their concerns deeply.
Collaborate: Involve all parties in the decision-making process to ensure they feel heard and respected.
Transparency: Clearly explain the rationale behind your decision and provide evidence supporting it.
Implement a Pilot: Suggest implementing a small-scale pilot for both approaches to validate the chosen path.

#### Closing Summary

In conclusion, the choice between event-driven choreography and a central orchestrator depends on the specific requirements of the order flow. By thoroughly analyzing performance, scalability, fault tolerance, and complexity, we can make an informed decision that aligns with our strategic goals. Ensuring open communication and collaboration will help maintain team morale and foster buy-in for the chosen approach.

#### Common Mistakes/Follow-Ups
Overlooking transactional requirements in favor of simplicity.
Ignoring future scalability needs when choosing a solution.
Failing to communicate transparently and inclusively, leading to resistance or dissatisfaction among stakeholders.

**Comparison:**

```
Architecture	Scalability	Fault Tolerance	Complexity
Event-Driven	High	Good	Low
Central Orch.	Moderate	Excellent	High
```

**Code:**

```
Example code for event-driven choreography (simplified)

def service_a_on_event(event): print(f"Received event {event}. Processing...") # Process the event

service_b.on_message(lambda message: service_c.process(message))

CODE
Copy

```python
# Example code for central orchestrator (simplified)
class Orchestrator:
    def handle_order(self, order_data):
        if self.validate(order_data):
            return self.execute_steps(order_data)
        else:
            raise ValueError("Invalid order data")

    def validate(self, order_data):
        # Validation logic

    def execute_steps(self, order_data):
        # Step-by-step execution
```

**Example input / output:**

```text
Input:
- Transaction volume: 1,000 per second
- Latency requirements: <500ms

Output:
- Event-driven choreography chosen due to better scalability and lower latency.
- Central orchestrator would be considered if higher transactional integrity is required.
```

**Data:**

```json
{
  "service_interactions": [
    {
      "service": "ServiceA",
      "events": ["OrderPlaced", "PaymentConfirmed"],
      "responses": []
    },
    {
      "service": "ServiceB",
      "events": ["PaymentRequestSent", "RefundRequested"],
      "responses": ["PaymentReceived", "RefundProcessed"]
    }
  ],
  "transaction_volume": 1000,
  "latency_requirements": 500,
  "error_handling_strategy": "Retry with exponential backoff"
}
```

**Flow:**

1. Event-Driven Choreography
2. Transaction Volume
3. Yes
4. Choose Event-Driven
5. No
6. Further Analysis Needed
7. Complexity of Workflows
8. Yes
9. Consider Central Orch.
10. No
11. Continue with Event-Driven
12. Central Orch. ConsideredBased on Specific Needs
13. Event-Driven Chosen forSimplicity and Scalability

