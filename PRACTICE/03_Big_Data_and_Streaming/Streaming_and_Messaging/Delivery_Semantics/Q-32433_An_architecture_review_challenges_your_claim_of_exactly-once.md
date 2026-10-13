---
type: "practice"
difficulty: "Expert"
---

# An architecture review challenges your claim of exactly-once. Design the chaos/verification test suite (dupe injection, kill-mid-commit, partition heal) that would actually prove or falsify the guarantee before you ship it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Exactly Once](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust verification test suite to validate exactly-once message delivery semantics in our architecture, we need to thoroughly understand the requirements and potential failure modes that could violate this guarantee. The key is to simulate realistic production scenarios where these failures might occur, such as dupe injection, mid-commit kill, and partition healing. We'll break down each of these test cases step by step using first-principles reasoning.

Firstly, let’s define the exactly-once semantics: ensure that a message is delivered exactly once to its intended consumer, without duplicates or omissions. This is critical for maintaining data integrity in distributed systems.

#### Triage

The question asks us to design tests that would prove or falsify the exactly-once guarantee. This involves understanding how these failures can occur and creating test cases to simulate those scenarios. We will assume a basic architecture with producers sending messages to brokers, which then route them to consumers via a reliable messaging protocol.

Key Assumptions
Message Broker Reliability: The broker is assumed to have mechanisms in place for durability (e.g., disk writes), but we need to test edge cases where these might fail.
Producer and Consumer Synchronization: We assume that producers can retry sending messages, while consumers might lose state or take time to process messages.
Test Cases
Dupe Injection
Objective: Verify that no duplicate messages are delivered to the consumer.
Test Setup: Send a message from a producer and verify it is not re-delivered after retries.
Kill-Mid-Commit
Objective: Simulate a scenario where a consumer is processing a message but is interrupted before committing its state.
Test Setup: Start a consumer, let it process a message partially, then stop the consumer and start again to check if the message is re-delivered or ignored.
Partition Heal
Objective: Verify that messages are not duplicated when a broker goes down and comes back up.
Test Setup: Bring down a broker, send messages while it’s offline, and then bring it back online; ensure these messages do not get re-delivered.

#### First-Principles Solution
Dupe Injection
Use message ID-based deduplication at the consumer level.
Implement a message broker that supports idempotent message delivery.
Write test cases where producers send identical messages with unique IDs to ensure they are only processed once.
Kill-Mid-Commit
Create a mock consumer that simulates partial processing of a message.
Introduce failure points after the consumer starts processing but before it commits its state.
Ensure the broker handles such failures by either re-delivering the message or marking it as in progress until committed.
Partition Heal
Simulate network partitions and broker restarts.
Verify that messages are not duplicated when a partition is healed.
Use logging and monitoring to track message delivery and ensure consistency.

#### Closing Summary

By designing these test cases, we can thoroughly verify the exactly-once guarantee in our architecture. This involves ensuring that no duplicate messages are delivered, even under partial processing or network partitions. The tests should simulate real-world failure scenarios to validate the reliability of our messaging system.

#### Common Mistakes/Follow-Ups
Failing to Account for Partial Processing: Ensure all test cases account for scenarios where a consumer might start but not complete message processing.
Over-reliance on Broker Mechanisms: Verify that the broker’s durability mechanisms are sufficient without completely relying on them, as they can fail in edge cases.
Neglecting Long-Term Data Integrity: Ensure long-term data integrity is maintained even if a consumer fails or goes offline for an extended period.

**Comparison:**

```
Test Case	Objective	Setup	Expected Outcome
Dupe Injection	Ensure no duplicate	Send identical messages with unique IDs	Messages are processed exactly once
Kill-Mid-Commit	Verify mid-commit logic	Simulate partial processing, stop consumer before commit	Consumer reprocesses only the message once
Partition Heal	Handle network partitions	Simulate broker failure and recovery	Messages are not duplicated during healing
```

**Code:**

```python
# Example test case for Dupe Injection using Python and a mock broker

import unittest
from mockbroker import MockBroker, MessageIdGenerator

class TestDupeInjection(unittest.TestCase):
    def setUp(self):
        self.broker = MockBroker()
        self.consumer_id_gen = MessageIdGenerator()

    def test_dupe_injection(self):
        # Send the same message with a unique ID
        msg1 = "Hello"
        id1 = next(self.consumer_id_gen)
        self.broker.send_message(id1, msg1)

        # Simulate consumer processing and check no duplicate messages are received
        response = self.broker.consume_message(id1)
        assert response == msg1

if __name__ == '__main__':
    unittest.main()
```

**Example input / output:**

```text
Input:
Producer sends message ID1 with content "Hello"

Expected Output:
Consumer receives message once, and its state is updated correctly.

Scenario - Kill-Mid-Commit:
Input:
Producer starts sending messages and Consumer begins processing but fails before committing.

Expected Output:
Consumer restarts and processes the same message only once.
```

**Data:**

```json
{
  "producer": {
    "message_id_gen": "MockMessageIdGenerator",
    "retry_attempts": 3,
    "retries_interval": 5000
  },
  "consumer": {
    "processing_timeout": 2000,
    "commit_interval": 1000
  }
}
```

**Flow:**

1. No
2. Yes
3. Producer sends message
4. Broker receives and routes
5. Consumer processes message
6. Commit state?
7. Re-send message to consumer
8. Consumer commits state

