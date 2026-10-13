---
type: "practice"
difficulty: "Intermediate"
---

# You must add a new 'partial refund' capability to a payments module without touching the hot 'capture' path; how do you find the extension seam and prove you didn't regress the capture flow?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Open-closed Extensibility](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Open-closed_Extensibility.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of adding a 'partial refund' capability to a payments module without impacting the critical 'capture' flow, we need to identify an extension point that allows us to introduce this new feature while maintaining the integrity and performance of the existing system. This is where low-level design (LLD) comes into play.

TL;DR

We will leverage a design pattern called "Command Bus" or "Event Sourcing" to find the right seam for adding the 'partial refund' capability. By doing so, we ensure that our changes do not affect the existing capture flow and can be validated through unit tests and integration tests.

#### Triaging the Question

The interviewer is probing how you would architect a feature addition in an existing system without causing regression issues, especially on critical paths like the 'capture' process. The key assumptions are:

There is an existing payments module with a 'capture' path that needs to remain untouched.
A new 'partial refund' capability must be added.

#### First Principles Solution
Identify the Seam: We need to find a logical place in the system where we can introduce changes without affecting the core functionality. This could be through event-driven architectures or by using design patterns like Command Bus, which allows us to handle commands asynchronously.
Command Bus Pattern: Implementing a Command Bus pattern will enable us to send 'Partial Refund' requests to an asynchronous queue. The existing 'capture' flow remains untouched because it processes only capture commands.
Event Sourcing: Use event sourcing to log all actions, including captures and partial refunds, ensuring that the state of the system can be reconstructed at any point in time.
Step-by-Step Solution

Define Commands and Events:

Define a PartialRefundRequest command.
Create corresponding PartialRefundRequestedEvent.

Command Handling:

Add a new handler for the PartialRefundRequest to an existing command bus, ensuring it does not interfere with existing handlers.

Event Processing:

Configure event processing logic that listens for the PartialRefundRequestedEvent, processes it, and updates the payment state accordingly.
Ensure this process is decoupled from the capture flow by using background workers or scheduled tasks to handle partial refunds.

Testing:

Write unit tests for the new command handler and event processing logic to verify they work as expected.
Perform integration testing to ensure that adding a 'partial refund' does not interfere with the existing 'capture' path.

#### Closing Summary

By employing patterns like Command Bus and Event Sourcing, we can extend the payments module without altering the critical capture flow. This approach ensures that any changes are isolated and validated through rigorous testing, thus maintaining the robustness of the existing system.

#### Common Mistakes/Follow-Ups
Failing to Isolate Changes: Not properly defining seams or isolating new features could lead to unintended side effects.
Insufficient Testing: Skipping thorough testing can result in regressions that only manifest later.
Overcomplicating the Solution: Adding unnecessary complexity without a clear need might make the system harder to maintain.

**Code:**

```
MERMAID
Copy
graph TD;
    A[User Issues Partial Refund] --> B(Dispatches PartialRefundRequest);
    B --> C(CommandBus);
    C --> D(EventProcessor)(Processes Event);
    D --> E(PaymentStateUpdater)(Updates State);
```

**Example input / output:**

```

Given an input PartialRefundRequest:

JSON
Copy
{
  "commandId": "12345",
  "paymentId": "67890",
  "amount": 10.50
}

The system will dispatch this command to the Command Bus, which will then trigger an PartialRefundRequestedEvent. This event is processed by the Event Processor, which updates the Payment State.
```

**Data:**

```json
{
  "PartialRefundRequest": {
    "commandId": "12345",
    "paymentId": "67890",
    "amount": 10.50
  }
}
```

