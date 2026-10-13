---
type: "practice"
difficulty: "Expert"
---

# A transactional-outbox poller and the app both write, and you get duplicate/out-of-order events. Walk through the concurrency guarantees the outbox does and doesn't give, and how you order/dedup on the consumer.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once via Idempotency](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Concurrency_Patterns/Exactly_Once_via_Idempotency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When implementing a transactional-outbox pattern in a distributed system, it's crucial to understand its concurrency guarantees and handle race conditions appropriately. The outbox ensures that events are written atomically but does not guarantee ordering or deduplication on the consumer side. To ensure ordered and unique event processing, you need to implement custom logic.

#### Triaging the Question

The question revolves around understanding how a transactional-outbox pattern operates in terms of concurrency guarantees and what mechanisms are needed for ordering and deduplication when consuming events from an outbox. The key points we'll cover include:

Concurrency Guarantees: What atomic operations does the outbox provide?
Ordering vs. Deduplication: How do you handle these concerns on the consumer side?

#### First-Principles Solution
Concurrency Guarantees of the Outbox

The transactional-outbox pattern typically ensures that a message is written atomically to a durable storage (like a database table) as part of a single transaction. This atomicity guarantees that once an event is committed, it cannot be rolled back or partially applied. However, this guarantee only applies within the scope of the transaction and does not extend beyond it.

Ordering and Deduplication on the Consumer Side
Ordering: Since the outbox itself does not provide any ordering guarantees, you need to implement custom logic in your consumer to ensure that events are processed in the order they were generated.
Deduplication: To handle duplicates, you must maintain a state of which messages have already been processed.
Step-by-Step Solution

Persistent State Management:

Use a separate database or cache to store metadata about consumed events (e.g., event IDs) as you process them.

Event Processing Pipeline:

Poll the outbox for new events in an ordered manner, ensuring that each poll respects the order of previous polls.

Duplicate Detection and Handling:

Before processing an event, check if it has been seen before using the persistent state management.
If a duplicate is detected, discard or skip the event without further processing.

Ordering Strategy:

Implement a mechanism to poll events in ascending order of their timestamps or sequence numbers (if available).

#### Example
Code

Below is an example implementation for deduplication and ordering:

PYTHON
import time

class OutboxPoller:
    def __init__(self, outbox_client, state_storage):
        self.outbox_client = outbox_client
        self.state_storage = state_storage

    async def poll_and_process_events(self):
        last_processed_id = await self.state_storage.get_last_processed_id()
        while True:
            # Poll for events greater than the last processed ID
            new_events = await self.outbox_client.fetch_events(last_processed_id)

            if not new_events:
                await self._update_state_with_processed_ids(new_events)
                continue

            for event in sorted(new_events, key=lambda x: x.id):
                if not await self.state_storage.mark_event_as_processed(event.id):
                    # Process the event
                    await self.handle_event(event)

    async def handle_event(self, event):
        # Implement your logic to process events here
        print(f"Processing event: {event}")

    async def _update_state_with_processed_ids(self, new_events):
        for event in new_events:
            await self.state_storage.add_processed_id(event.id)

#### Closing Summary
and Common Mistakes

To ensure that you handle concurrency correctly in a distributed system using an outbox pattern, it's crucial to implement custom ordering and deduplication logic. The outbox itself provides atomic writes but not order guarantees or deduplication. By maintaining a persistent state of processed events and polling for new ones in the correct order, you can avoid race conditions and ensure that your consumer behaves as expected.

#### Common Mistakes
Overreliance on Outbox Guarantees: Assuming the outbox handles ordering and deduplication without additional code.
Lack of State Management: Not maintaining a state of processed events, leading to duplicate processing or missed events.
Polling Strategy: Using an inefficient polling strategy that doesn't respect the order or misses events.

**Comparison:**

```
Event ID	Timestamp
12345	1672508400
12367	1672508410
12389	1672508420
```

**Code:**

```python
# Example implementation in Python
```

**Example input / output:**

```text
Input:
- Fetch events from outbox with last_processed_id = "12345"
Output:
- [Event with id="12367", Event with id="12389"]
```

**Data:**

```json
{
  "event": {
    "id": "12345",
    "timestamp": 1672508400,
    "data": {}
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Start Polling
5. Check Last Processed ID
6. Fetch Events from Outbox
7. Process New Events?
8. Mark Event as Processed andStore State
9. Continue Polling
10. Discard Duplicate Event

