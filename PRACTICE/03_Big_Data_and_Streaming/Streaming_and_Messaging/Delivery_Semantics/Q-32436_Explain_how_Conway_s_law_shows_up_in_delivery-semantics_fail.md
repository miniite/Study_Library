---
type: "practice"
difficulty: "Deep"
---

# Explain how Conway's law shows up in delivery-semantics failures: when producer and consumer teams disagree on the contract, duplicates and losses emerge at the seam. Design the team-topology and contract-testing approach to fix the human root cause.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Dedup Contracts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Conway's Law is a principle stating that the organizational structure of a company will influence the design of its software systems. In the context of message delivery semantics, this law often manifests as misaligned expectations between producer and consumer teams, leading to issues like duplicates or losses in the system. This mismatch typically occurs at the "seam" where these two teams interact. To address these failures, we need a deep understanding of how to design team topologies that promote clear communication and consistent contract testing.

Triaging the Problem

The core issue is about ensuring that both producer and consumer teams agree on the semantics of message delivery—whether it should be "at-least once" or "exactly once." When these expectations diverge, synchronization issues can arise. The goal is to design a system where both sides operate under mutual agreement, reducing the risk of duplicates or losses.

#### Assumptions
and Context
Producer Team: Responsible for sending messages.

#### Cons
umer Team: Responsible for processing messages.
Message Delivery Semantics: Agreed-upon behavior when messages are sent and received (e.g., once-only delivery).
Team Interactions: The point of interaction where producer sends a message, and consumer processes it.

#### First Principles Solution
Define Clear Contracts: Establish well-defined interfaces between the producer and consumer teams.
Implement Contract Testing: Use automated tests to ensure both sides adhere to these contracts.
Align Team Boundaries: Organize teams so that they have a shared understanding of the system's requirements.
Step-by-Step Solution

Define Clear Contracts:

Define message formats and semantics in terms that both teams understand.
Use well-defined APIs or protocols for communication.

#### Example
If "exactly once" delivery is required, ensure messages are idempotent and the consumer can handle retries without processing duplicates.

Implement Contract Testing:

Both producer and consumer teams should write tests to validate that they adhere to the agreed-upon contract.
Use integration testing frameworks to simulate interactions between components.

#### Example
A test might verify that a message is delivered exactly once by ensuring the same identifier does not trigger multiple processing runs.

Align Team Boundaries:

Ensure both teams are colocated or have frequent communication channels.
Establish regular review meetings where teams discuss and align on contract details.

#### Example
Weekly sync sessions to revisit and update contracts as necessary.

Design for Human Interaction:

Foster a culture of collaboration and shared ownership over the system's behavior.
Implement practices like pair programming or code reviews across both teams.

#### Example
Encourage developers from both sides to work on each other’s code, gaining mutual understanding.

#### Closing Summary

By clearly defining contracts, implementing robust testing strategies, aligning team boundaries, and fostering strong human interactions, we can mitigate the risks associated with mismatched expectations between producer and consumer teams. This approach ensures that both sides operate under a shared understanding of how messages should be delivered and processed, thereby reducing duplicates and losses.

#### Common Mistakes/Follow-Ups

Mistake: Assuming one team's perspective is inherently correct.

#### Follow-Up
Both teams need to engage in equal dialogue to ensure mutual understanding.

Mistake: Failing to establish or test contracts rigorously.

#### Follow-Up
Use formal methods like BDD (Behavior Driven Development) to define and validate requirements.

**Comparison:**

```
Test Case	Scenario	Expected Result
1	Send a single unique message	Processed exactly once
2	Send duplicate messages	Throws error
3	Retry processing with same ID	Error on retry attempt
```

**Code:**

```python
# Example of a simple contract test for "exactly once" delivery
def test_exactly_once_delivery():
    # Mock message sending by producer
    mock_producer.send(message="test_message")

    # Consumer processing logic (simplified)
    def process_message(message):
        if message == "test_message":
            raise ValueError("Duplicate message detected")
        return True

    # Simulate consumer processing with retries
    for _ in range(5):
        if not process_message(mock_producer.get_next_message()):
            break  # If duplicate is detected, break the loop

# Output: No errors, indicating "exactly once" delivery is maintained.
```

**Example input / output:**

```text
Input (Producer):
- Send a message with ID P123 to the consumer.

Output (Consumer):
- Process the message once.
- If duplicate, throw an error: "Duplicate message detected"

Expected Outcome:
- Consumer processes message exactly once without any errors.
```

**Data:**

```json
{
  "producer": {
    "id": "P123",
    "message": "test_message"
  },
  "consumer": {
    "id": "C456",
    "processing_logic": "def process_message(message): if message == 'test_message': raise ValueError('Duplicate message detected') return True"
  }
}
```

**Flow:**

1. Yes
2. No
3. Producer Sends Message
4. Is Consumer Ready?
5. Consumer Processes Message
6. Send Acknowledgment toProducer
7. Reschedule Send AttemptLater

