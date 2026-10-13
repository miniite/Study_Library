---
type: "practice"
difficulty: "Deep"
---

# How do you keep a saga idempotent so a retried step doesn't double-book or double-refund?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read Your Writes in Event Systems](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Read_Your_Writes_in_Event_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-driven architecture, ensuring that a saga remains idempotent is crucial to avoid issues like double-booking or double-refunding when steps are retried. A saga in this context represents a long-running business process split into individual steps, each of which may be asynchronous and processed by different services via message queues. To achieve idempotency, we need to ensure that any step can be retried without causing unintended side effects.

This problem is essentially about handling duplicate messages or retries gracefully. The core idea involves using unique identifiers for sagas, ensuring that intermediate states are tracked, and implementing compensating actions where necessary. Let's break this down from first principles.

#### Triage

The question revolves around maintaining the idempotency of a saga in an event-driven architecture. Specifically, it seeks to prevent double-booking or double-refunding when retries occur due to failures. The key is ensuring that steps are retried correctly without leading to duplicate processing.

#### Assumptions
and First-Principles Solution
Saga Management: We manage sagas using unique identifiers.
State Tracking: Each step of the saga has a known state, which we can use to determine if it has already been processed.
Compensating Actions: If an action needs to be undone due to a failure or retry, there should be a way to revert the changes made by that action.
Step-by-Step Solution

Saga Identifier and State Tracking

Assign each saga a unique identifier (SAGA_ID) at its inception.
Track the state of each step in the saga using a database or distributed ledger, like DynamoDB or Kafka Streams.

Message Processing with Idempotency Key

When processing messages from queues, include an idempotency key that is derived from the SAGA_ID and possibly some context (like transaction ID) to ensure uniqueness.
Use this key to check if the message has already been processed before executing any steps.

Compensating Actions

For each step in a saga, define compensating actions that can reverse the effects of that step if it fails or needs to be retried.
Ensure these compensating actions are idempotent as well, meaning they can be run multiple times without causing additional side effects.

Handling Retries

When a message is retried due to an error, use its idempotency key to check if it has already been processed.
If the step has not been processed yet or has been marked as in progress (using optimistic locking), proceed with the execution and mark it as completed.
If the step has already been processed, simply return success without executing it again.

Error Handling and Logging

Implement robust error handling to capture and log failures at each step.
Ensure that errors are properly propagated so retries can be managed effectively.

#### Closing Summary

To keep a saga idempotent in an event-driven architecture, we need to ensure unique identifiers for each saga instance, track the state of each step, use idempotency keys for message processing, and implement compensating actions. This approach prevents double-booking or double-refunding by ensuring that any step can be retried without causing unintended side effects.

#### Common Mistakes/Follow-Ups
Failing to Track State: Not maintaining a state in a database or distributed ledger can lead to duplicate processing.
Idempotency Key Misuse: Incorrectly using idempotency keys can result in retrying processes that should not be retried.
Lack of Compensating Actions: Failing to define and implement compensating actions can result in data inconsistencies if steps are retried.

**Comparison:**

```
Step	Execution Status
Book	Completed
Refund	In Progress
```

**Code:**

```python
# Example Python code for saga management
class Saga:
    def __init__(self, saga_id):
        self.saga_id = saga_id
        self.step_states = {}

    def process_step(self, step_name, idempotency_key):
        if idempotency_key not in self.step_states or self.step_states[idempotency_key] == "in_progress":
            # Mark as in progress and execute the step
            self.step_states[idempotency_key] = "processing"
            result = execute_step(step_name)
            self.step_states[idempotency_key] = "completed"
            return result
        else:
            # Step has already been processed, do nothing
            return True

def execute_step(step_name):
    # Simulate step execution and error handling
    if step_name == "book":
        # Book a transaction
        book_transaction()
        return "Booked successfully"
    elif step_name == "refund":
        # Refund a transaction
        refund_transaction()
        return "Refunded successfully"
```

**Example input / output:**

```text
Input:
-Saga ID: SAGA-12345
-Step Name: book

Output:
"Booked successfully"
```

**Data:**

```json
{
  "saga_id": "SAGA-12345",
  "steps": [
    { "step_name": "book", "idempotency_key": "BK-67890" },
    { "step_name": "refund", "idempotency_key": "RF-67891" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Receive Message
4. Is Idempotency Key Known?
5. Execute Step
6. Maintain State
7. Mark as In Progress & ExecuteStep
8. Complete Step & Mark asCompleted

