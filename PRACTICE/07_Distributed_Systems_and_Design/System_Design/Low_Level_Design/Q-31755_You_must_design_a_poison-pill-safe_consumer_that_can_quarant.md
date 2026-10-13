---
type: "practice"
difficulty: "Deep"
---

# You must design a poison-pill-safe consumer that can quarantine a bad message but keep processing later ones without losing at-least-once guarantees; how do you model offset commit vs quarantine so nothing is lost or skipped?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Compensating Transactions](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Compensating_Transactions/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this low-level design (LLD) challenge, we need to ensure that our consumer can handle bad messages while still maintaining at-least-once processing guarantees. The core idea here involves carefully managing offsets and quarantining bad messages without losing any valid ones. Let's break down the problem from first principles.

First, we must understand what "poison-pill-safe" means in this context: it implies that our consumer should be able to handle erroneous messages (poison pills) and isolate them from the main processing flow without affecting the overall correctness of the system. To achieve this, we need to balance between offset commit strategies and quarantine mechanisms.

#### Triaging the Question

The interviewer is probing:

Understanding of At-Least-Once Processing: This means every message should be processed at least once.
Handling Poison Pills: Bad or erroneous messages that could disrupt the processing flow.
Offset Management Strategies: How to commit offsets and ensure they are consistent with the current state of processing.

#### Assumptions
The consumer uses a message broker like Kafka, which supports offset management for at-least-once delivery.
A poison pill is any erroneous or malformed message that could cause issues in the processing logic.
Quarantine refers to isolating bad messages from the main flow of processing.

#### Solution
First Principles

To solve this problem, we need to:

Isolate Bad Messages: Temporarily store or quarantine these messages without affecting the processing of subsequent valid ones.

#### Cons
istent Offset Management: Ensure that offsets are committed only after all messages up to a certain point have been processed and successfully stored in the system.
Step-by-Step Solution

Message Processing Flow:

Each message is processed by the consumer.
If the message is valid, it's processed normally, and an offset is committed to mark that message as handled.
If a message is detected as bad (poison pill), it’s quarantined, but the processing of subsequent messages continues.

Offset Commit Strategy:

We can use a strategy where offsets are only committed after all messages up to a certain point have been successfully processed and stored in a reliable store.
This ensures that even if there is a failure between the commit and the next message, no valid messages will be lost.

Quarantine Mechanism:

Define a quarantine mechanism where bad messages are stored separately but do not affect the processing of other messages.
Ensure that any error in handling a message does not cause an offset to be committed prematurely or skipped.

Handling Quarantined Messages:

After processing all valid messages, we can review and handle quarantined messages at our convenience without affecting the processing order of new incoming messages.

#### Closing Summary

In summary, to design a poison-pill-safe consumer that processes bad messages while maintaining at-least-once guarantees:

We use a quarantine mechanism to isolate bad messages.
Offsets are committed only after all messages up to a certain point have been successfully processed and stored.
This ensures no valid message is lost or skipped, providing robust handling of erroneous inputs.

#### Common Mistakes/Follow-Ups

Common pitfalls include:

Failing to properly handle offsets and ensuring consistency.
Not isolating bad messages effectively, leading to corrupted state in the system.
Prematurely committing offsets that could cause data loss if a failure occurs after commit but before processing all messages.

#### Follow-ups
might involve discussing how this solution scales with increasing message volumes or handling specific types of poison pills (e.g., network failures vs. application logic errors).

**Comparison:**

```
Message ID	Status
123	processed
456	quarantined
```

**Code:**

```python
# Example pseudocode for offset management and quarantine

def process_message(message, consumer):
    try:
        # Process the message
        result = process_logic(message)
        if result.is_valid():
            # Commit offset only after successful processing and storage
            consumer.commit_offset()
        else:
            # Quarantine the bad message
            store_quarantined_message(message)
    except Exception as e:
        handle_exception(e)

def handle_exception(exception):
    log.error(f"Error processing message: {exception}")
```

**Example input / output:**

```text
Input:
- Message "123" -> Valid
- Message "456" -> Invalid

Output:
- Processed message "123"
- Quarantined message "456" but no offset committed prematurely
```

**Data:**

```json
{
  "message": {
    "id": "123",
    "data": "valid_data",
    "status": "processed"
  },
  "quarantined_message": {
    "id": "456",
    "data": "bad_data",
    "status": "quarantined"
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Message
4. Is Valid?
5. Process and Commit Offset
6. Quarantine Message
7. Next Message
8. Review Quarantined MessagesLater

