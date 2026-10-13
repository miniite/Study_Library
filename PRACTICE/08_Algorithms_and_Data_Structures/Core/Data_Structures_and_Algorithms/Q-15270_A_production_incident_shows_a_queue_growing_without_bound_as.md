---
type: "practice"
difficulty: "Deep"
---

# A production incident shows a queue growing without bound as consumers fall behind; how do you decide between backpressure, dropping, and load-shedding, and what does each imply for the data structure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Bounded Blocking Queues](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Bounded_Blocking_Queues.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this production issue, we need to understand three key strategies: backpressure, dropping, and load-shedding. Each approach has its own implications on how the queue is managed and impacts the overall system architecture.

Backpressure involves signaling consumers that they are falling behind by reducing their rate of processing or even halting them altogether. Dropping means simply discarding items from the queue when consumers cannot keep up. Load-shedding is about selectively dropping older, less critical messages to free up resources for more urgent tasks. These strategies can be implemented differently depending on the data structures and algorithms used.

**Comparison:**

```
Strategy	Signal to Consumers	Action Taken on Queue	Impact on System
Backpressure	Yes	Reduce processing rate	Slows system down but keeps data
Dropping	No	Discard items	May lose data
Load-shedding	No	Remove non-critical tasks	Balances resource usage
```

**Code:**

```
Example of implementing backpressure using async signals

import asyncio

async def consumer(tasks): while True: task = await asyncio.wait_for(asyncio.current_task().loop.get_running_loop().run_in_executor(None, tasks.get), timeout=0.1) if not task.done(): task.cancel()

CODE
Copy

```python
# Example of implementing dropping and load-shedding
def drop_oldest_queue(queue):
    return queue.popleft()  # Simple FIFO queue

def shed_load_dropping_critical(queue, critical_items):
    for item in reversed(list(queue)):
        if not any(item == crit_item for crit_item in critical_items):
            queue.remove(item)
```

**Example input / output:**

```text
Input:
- Queue: `["task1", "task2", "task3", "task4"]`
- Critical tasks: `["task3"]`

Output for dropping (oldest first):
- Resulting queue after drop_oldest_queue(queue): `["task2", "task3", "task4"]`

Output for load-shedding:
- Resulting queue after shedding non-critical tasks: `["task3"]`
```

**Data:**

```json
{
  "queue": ["task1", "task2", "task3", "task4"],
  "critical_tasks": ["task3"]
}
```

**Flow:**

1. Queue Full
2. Backpressure: Slow downconsumers
3. Dropping: Discard items
4. Load-shedding: Drop lesscritical tasks

