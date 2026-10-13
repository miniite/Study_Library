---
type: "practice"
difficulty: "Deep"
---

# How would you compute trending items over a sliding time window efficiently?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Clickstream Counting Pipeline](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute trending items over a sliding time window efficiently, we need to design a system that can handle high throughput and maintain low latency while ensuring accuracy in tracking item popularity. The core challenge is to update our rankings as new data arrives and old data expires within the sliding window without recomputing everything from scratch.

The question is essentially probing your understanding of efficient algorithms and data structures, as well as your ability to balance time complexity with space efficiency. We’ll assume a few key things: the items are typically strings or integers, the system processes millions of events per second, and we’re dealing with a fixed-size sliding window (e.g., 30 minutes).

Let's break this down step-by-step.

#### First Principles Solution

Define the Problem:

We need to maintain a list of items where their popularity is computed over a sliding time window.
The time window has a fixed duration, say ( W ) seconds.
Each event consists of an item and its timestamp.

Data Structures:

Use a priority queue (min-heap) to keep track of the top trending items efficiently by their popularity scores.
Maintain a hash map to quickly look up and update the frequency of each item within the sliding window.
A doubly linked list can be used to manage the time decay, ensuring that old events are dropped as they expire from the current window.

Algorithm:

For every new event, update its timestamp in the hash map and adjust its position in the heap based on updated frequency.
Use a timer or clock to automatically remove expired items from both the hash map and the doubly linked list.
Periodically recompute the heap if necessary to maintain accuracy.

#### Implementation
Details:

Ensure efficient insertion and deletion operations in the heap (logarithmic time).
Hash map provides O(1) lookups and updates, critical for high throughput.
Doubly linked list allows efficient removal of expired entries.
Detailed Steps
1. Initialize Data Structures
Hash Map: Maps each item to its frequency within the sliding window.
Min Heap: Stores items based on their popularity score (frequency).
Doubly Linked List: Manages temporal order, ensuring that older events are removed first when the window slides.
2. Process Each Event

Update Frequency:

Increment the frequency of the item in the hash map.

Maintain Sliding Window:

Check if the current time has exceeded the sliding window duration.
Remove any items from both the heap and hash map that are now outside the window.

Recompute Top Items:

Periodically (e.g., every 5 minutes), recompute the top trending items by popping elements from the min-heap until we have the desired number of items.
3. Example Dry Run
PYTHON
# Sample Event: {"item": "apple", "timestamp": 1628409600} # Unix timestamp
from heapq import heappush, heappop

events = [
    {"item": "apple", "timestamp": 1577836800},
    {"item": "orange", "timestamp": 1577836800 + 3600}, # 1 hour later
    {"item": "banana", "timestamp": 1577840400}       # 2 hours later
]

hash_map = {}
heap = []
linked_list = []

for event in events:
    item, timestamp = event["item"], event["timestamp"]

    if item not in hash_map:
        hash_map[item] = {"frequency": 1}
        heappush(heap, (-hash_map[item]["frequency"], item))
    else:
        old_freq = hash_map[item]["frequency"]
        new_freq = old_freq + 1
        if timestamp - linked_list[0][1] > 3600: # Sliding window size in seconds
            heap.remove((-old_freq, item))
            heappush(heap, (-new_freq, item))

    # Adjust linked list and hash map for sliding window maintenance

# Output the top trending items from the heap
while len(heap) > 3:
    heappop(heap)
print([item for freq, item in heap])

#### Closing Summary
+ Common Mistakes/Follow-Ups
The key idea is to use a combination of data structures (heap and hash map) to maintain both efficient updates and queries.

#### Common mistakes
include not properly managing the sliding window boundaries or using inefficient data structures that lead to poor performance.

#### Follow-ups
could involve discussing trade-offs between space complexity, time complexity, and accuracy in trending computations. Another follow-up might be to discuss how this system would scale with increasing event rates or item counts.

**Comparison:**

| Time Window | Item   | Frequency |
|-------------|--------|-----------|
| 1628409600-1628431200 | apple  | 1         |
| 1628409600-1628431200 | orange | 1         |
| 1628409600-1628431200 | banana | 0         |

**Code:**

```python
# Example code snippet for dry run
from heapq import heappush, heappop

events = [
    {"item": "apple", "timestamp": 1577836800},
    {"item": "orange", "timestamp": 1577836800 + 3600}, # 1 hour later
    {"item": "banana", "timestamp": 1577840400}       # 2 hours later
]

hash_map = {}
heap = []

for event in events:
    item, timestamp = event["item"], event["timestamp"]

    if item not in hash_map:
        hash_map[item] = {"frequency": 1}
        heappush(heap, (-hash_map[item]["frequency"], item))
    else:
        old_freq = hash_map[item]["frequency"]
        new_freq = old_freq + 1
        if timestamp - linked_list[0][1] > 3600: # Sliding window size in seconds
            heap.remove((-old_freq, item))
            heappush(heap, (-new_freq, item))
```

**Example input / output:**

```text
Input:
[
    {"item": "apple", "timestamp": 1577836800},
    {"item": "orange", "timestamp": 1577836800 + 3600}, # 1 hour later
    {"item": "banana", "timestamp": 1577840400}       # 2 hours later
]

Output:
["apple", "orange"]
```

**Data:**

```json
[
    {"item": "apple", "timestamp": 1577836800},
    {"item": "orange", "timestamp": 1577836800 + 3600}, # 1 hour later
    {"item": "banana", "timestamp": 1577840400}       # 2 hours later
]
```

**Flow:**

1. Receive Event
2. Update Frequency in HashMap
3. Maintain Sliding WindowBoundaries
4. Recompute Top ItemsPeriodically
5. Return Top Trending Items

