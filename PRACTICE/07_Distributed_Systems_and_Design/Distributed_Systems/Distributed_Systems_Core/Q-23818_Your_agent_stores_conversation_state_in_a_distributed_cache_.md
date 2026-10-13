---
type: "practice"
difficulty: "Intermediate"
---

# Your agent stores conversation state in a distributed cache and a cache node failover loses a few seconds of recent turns, so the agent 'forgets' mid-conversation. Walk me through making conversation state durable without adding unacceptable per-turn latency.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Slow Dependencies and Load Shedding](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Deadline_Propagation/Slow_Dependencies_and_Load_Shedding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of losing recent conversation turns during cache node failover, we need to ensure that our conversation state is resilient and durable across nodes in a distributed system. The goal is to maintain the conversation history without introducing unacceptable latency per turn. Here’s how we can achieve this:

First, let's summarize the key challenge: a node failure results in loss of recent conversation data, causing the agent to 'forget' parts of the conversation mid-way.

#### Triage

The question is essentially asking for strategies to ensure that all conversation state updates are persisted reliably before they can be acknowledged by our application. This involves understanding the nature of distributed caching and implementing a mechanism that ensures data consistency across nodes without significant latency impacts.

#### Assumptions
The system uses a distributed cache (e.g., Redis or Memcached).
Each turn in the conversation is represented as a state update.
The agent needs to acknowledge receipt of each turn before moving forward with the next one.

#### Solution

We can address this by using a combination of durable storage and coordination mechanisms. Here’s a step-by-step solution:

Use Append-Only Storage: Implement an append-only log (AOL) mechanism, such as a write-ahead log (WAL). This ensures that every update to the conversation state is written to a durable storage before it's acknowledged by the cache node.
Cache Updates and Acknowledgment: When an agent receives a new turn in the conversation, it first writes this state to the append-only log. Once the write operation succeeds (indicating durability), the agent then updates the distributed cache. If the cache node fails during this process, the client can retry the update from the WAL.
Coordination Mechanisms: Use coordination tools like distributed locks or consensus algorithms (e.g., Raft) to manage writes and ensure that only one writer updates the state at a time.

#### Example
Workflow
Agent receives new conversation turn: "User asks about weather in London."
Agent logs this update to the append-only log.
The agent then attempts to update the cache node with the latest state (conversation history up to the current turn).
If the cache node fails before confirming the update, the agent can retry from the WAL.

#### Closing Summary

By combining durable storage with coordination mechanisms, we can ensure that conversation state is resilient against node failures while maintaining low latency during normal operations. The key steps involve logging updates to an append-only log and using this as a source of truth for cache updates.

#### Common Mistakes/Follow-Ups
Ignoring Durability: Failing to use durable storage mechanisms like WAL.
Inadequate Retries: Not handling retries correctly, which can lead to inconsistent states.
Overhead from Coordination: Ensuring that coordination tools (like distributed locks) do not introduce significant overhead.

**Comparison:**

```
Turn	Action
1	Write to WAL
2	Cache update attempt
3	Handle cache failure
```

**Code:**

```python
class ConversationManager:
    def __init__(self, cache_client):
        self.cache_client = cache_client
        self.log = []

    def receive_turn(self, turn_data):
        # Step 1: Write to the append-only log (WAL)
        self.log.append(turn_data)

        # Step 2: Update the distributed cache
        try:
            self.cache_client.update_conversation_state(turn_data)
        except CacheFailureException:
            # Log error and retry from WAL on failure
            pass

    def handle_cache_failure(self):
        for turn in reversed(self.log):
            try:
                self.cache_client.update_conversation_state(turn)
                break  # Stop once the state is successfully updated
            except CacheFailureException:
                continue
```

**Example input / output:**

```

Input:

Agent receives turn: "User asks about weather in London."

Output:

Append to WAL: {"turn_data": {"user_turn": "User asks about weather in London.", "agent_turn": "The current weather is sunny and 25°C."}}
Try updating cache node.
Cache node fails, retry from WAL.

Dry-run:

PYTHON
Copy
>>> manager.receive_turn({"user_turn": "User asks about weather in London.", "agent_turn": "The current weather is sunny and 25°C."})
```

**Data:**

```json
{
    "turn_data": {
        "user_turn": "User asks about weather in London.",
        "agent_turn": "The current weather is sunny and 25°C."
    }
}
```

**Flow:**

1. Success
2. Failure
3. Receive New Turn
4. Write to Append-Only Log(WAL)
5. Try Update Cache Node
6. Acknowledge and Proceed
7. Retry from WAL
8. Handle Retry Attempts
9. | Key | Value |
10. |-----|-------|
11. | Correctness + Completeness | Confirmed |

