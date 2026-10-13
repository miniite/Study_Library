---
type: "practice"
difficulty: "Expert"
---

# Explain how consumer group rebalancing under at-least-once can produce duplicate processing even with perfectly correct code, and design the fencing/generation-id strategy to make reprocessing safe.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Dedup Contracts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Dedup_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with message delivery semantics in a distributed system, ensuring that each message is processed at least once is critical but can introduce complexity, especially during consumer group rebalancing. This scenario often arises in systems using Kafka or similar platforms, where messages might be consumed by multiple consumers within the same consumer group. During rebalancing, if not managed properly, duplicate processing can occur due to messages being delivered more than once.

#### Triage

The question is asking us to understand how perfectly correct code can still lead to duplicate message processing during consumer group rebalancing in an at-least-once delivery semantic system. The core of the problem lies in the asynchronous nature of rebalancing and the potential overlap in message consumption between old and new consumer sets.

We need to design a fencing or generation ID strategy that ensures reprocessing is safe, meaning duplicate messages are only processed once even when the same message is delivered more than once due to rebalancing. This involves understanding the interactions between partitions, offsets, and consumer states during rebalancing.

#### First-Principles Solution
Partitioning and Offsets: In a distributed system like Kafka, messages are distributed across partitions within topics. Each partition has an offset that tracks the position of the last processed message.

#### Cons
umer Group Management: A consumer group is responsible for consuming all messages from its assigned partitions. When a rebalance occurs, the consumer group redistributes these partitions among its consumers.
Fencing and Generation IDs:
Fencing: This strategy ensures that only one instance of a consumer in a partition processes any given message to prevent duplicates.
Generation ID Strategy: By assigning a unique generation ID to each rebalance event, we can track which messages belong to the current processing context.
Detailed Steps

Offset Management:

Maintain a list of offsets for each partition that are already processed during the previous or ongoing consumer group operation.
During rebalancing, update this list based on the new assignments and current offsets.

Fencing Mechanism:

Implement a mechanism to detect if a message is being processed again in another instance within the same partition.
Use a lock-based approach where only one instance of the consumer can process messages from a specific offset range at any given time.

Generation ID Strategy:

Assign a unique generation ID to each rebalance event and update it as new partitions are assigned or retracted.
Messages processed with an outdated generation ID should be marked for safe reprocessing only if their offsets are not in the current processing list.

Safe Reprocessing:

Ensure that messages with matching offsets and correct generation IDs are retried only once, even if they were delivered multiple times due to rebalancing.
This can involve maintaining a global or local state for tracking processed messages and their associated generation IDs.

#### Closing Summary

In summary, ensuring safe reprocessing in an at-least-once delivery system during consumer group rebalancing requires meticulous management of offsets, fencing mechanisms, and generation IDs. By implementing these strategies, we can prevent duplicate processing while still guaranteeing that all necessary messages are delivered and consumed at least once.

#### Common Mistakes
and Follow-Ups
Mistake 1: Ignoring the asynchronous nature of rebalancing, leading to missed or double-processing of messages.
Mistake 2: Not using generation IDs correctly, which can result in undetected duplicate processing.

#### Follow-Up
Discussing trade-offs between performance and reliability, such as whether to use per-partition locks versus global state management.

**Comparison:**

```
Offset	Generation ID
0	gen1
1	gen2
2	gen4
```

**Code:**

```python
def process_message(message, offsets, current_generation_id):
    if message.offset not in processed_messages or message.generation_id == current_generation_id:
        # Process the message safely
        handle_message(message)
        processed_messages.add(message.offset)
```

**Example input / output:**

```
Input: Message with offset=2 and generation id="gen4", offsets=[0, 1, 2], current_generation_id="gen3".
Output: Process message safely as its offset is not in processed_messages and matches the current generation ID.
Dry-Run:
PYTHON
Copy
message = {"offset": 2, "generation_id": "gen4"}
current_generation_id = "gen3"
offsets = [0, 1, 2]
processed_messages = [[0, "gen1"], [1, "gen2"]]

if message["offset"] not in processed_messages or message["generation_id"] == current_generation_id:
    print("Processing message safely")
else:
    print("Message already processed")
```

**Data:**

```json
{
  "offsets": [0, 1, 2],
  "processed_messages": [[0, "gen1"], [1, "gen2"]],
  "generation_id": "gen3"
}
```

**Flow:**

1. Rebalance Starts
2. Consumer AssignmentsChange
3. Update Offsets and GenerationID
4. Fence New MessageProcessing
5. Safe Reprocessing Check

