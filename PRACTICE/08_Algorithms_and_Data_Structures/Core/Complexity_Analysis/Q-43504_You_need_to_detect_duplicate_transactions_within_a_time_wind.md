---
type: "practice"
difficulty: "Expert"
---

# You need to detect duplicate transactions within a time window at high TPS; how do you reason about the time and space complexity of a keyed sliding-window store versus a Bloom filter with a rotating window?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sorting Cost in Services](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Cost_in_Services.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the problem of detecting duplicate transactions within a time window, we need to compare two storage strategies: a keyed sliding-window store and a Bloom filter with a rotating window. Both methods have their unique trade-offs in terms of time and space complexity.

First, let's break down the requirements. We are dealing with high transaction per second (TPS) rates, which means efficient processing is critical. The goal is to ensure that we can quickly identify duplicates within a specific time window while minimizing both time and space overheads.

Triaging the Problem
Keyed Sliding-Window Store: This approach involves maintaining a data structure where each transaction key has an associated timestamp. As transactions come in, we check if they are within the current time window.
Bloom Filter with Rotating Window: A Bloom filter is used to quickly test whether an element might be a member of a set. We rotate the window by removing old entries and adding new ones as transactions arrive.

#### Assumptions
The transaction IDs are unique identifiers.
The time window size ( W ) and the TPS rate are constants for simplicity in our analysis.
We use a hash function to map transaction keys to indices for both methods.
Keyed Sliding-Window Store
Time Complexity

The primary operation is checking if a transaction exists within the current window. In a well-implemented sliding-window store, this can be done in constant time ( O(1) ). However, inserting and removing transactions require additional operations:

Insert: Amortized ( O(\log n) ) due to potential rehashing.
Remove: ( O(\log n) ), similar to insertion.
Space Complexity

The space required grows linearly with the number of unique transaction IDs in the current window. If we have a high TPS, this can quickly become large.

Bloom Filter with Rotating Window
Time Complexity
Insertion: ( O(1) ), as each element is hashed and checked.
Checking for Duplicates: Also ( O(1) ).

Since both operations are constant time, the overall efficiency is high. However, there's a risk of false positives due to the nature of Bloom filters.

Space Complexity

The space required by a Bloom filter depends on the desired accuracy (false positive rate). Typically, for a reasonable error rate, we need: [ \text{Space} = O\left(\frac{\log p}{1 - p}\right) n ] where ( n ) is the number of insertions and ( p ) is the false positive probability.

Full Solution
Keyed Sliding-Window Store

To implement this, we can use a hash table where each key (transaction ID) maps to a timestamp. When a transaction comes in:

Check if it's within the current window.
If not, check if its timestamp is older than any existing entry.
Remove old entries outside the window.

This approach ensures exact duplicates but may use more space as the number of unique transactions grows over time.

Bloom Filter with Rotating Window

To implement this:

Initialize a Bloom filter for each new time window.
As transactions come in, hash them and check or insert into the appropriate Bloom filter.
Periodically clear old filters to maintain efficiency.

This approach trades space for speed but risks false positives, which might not be acceptable depending on the application's requirements.

#### Closing Summary

In summary, a keyed sliding-window store provides precise duplicate detection at the cost of higher space usage, while a Bloom filter with a rotating window offers faster processing at the expense of potential false positives. The choice depends heavily on the specific application constraints and desired trade-offs between accuracy and efficiency.

#### Common Mistakes/Follow-Ups
Ignoring Space Constraints: Failing to consider that high TPS can quickly exhaust space in a sliding-window store.
Not Accounting for False Positives: Assuming Bloom filters are always acceptable, even when false positives could be problematic.
Overlooking Window Management: For both methods, effective management of old data is crucial but often overlooked.

**Comparison:**

| Method                  | Time Complexity | Space Complexity |
|-------------------------|-----------------|------------------|
| Sliding Window Store    | O(1)            | O(n * window_size) |
| Bloom Filter with Rotating Window | O(1)           | O(window_size / error_rate * n) |

**Code:**

```
Example code for a simple sliding-window store

class SlidingWindowStore: def init(self, window_size): self.window_size = window_size self.timestamp_map = {}

CODE
Copy
def add_transaction(self, transaction_id, timestamp):
    if len(self.timestamp_map) > self.window_size * 2:  # Double the size for safety
        oldest_key = min(self.timestamp_map.keys(), key=lambda k: self.timestamp_map[k])
        del self.timestamp_map[oldest_key]

    self.timestamp_map[transaction_id] = timestamp

def has_duplicate_in_window(self, transaction_id):
    return transaction_id in self.timestamp_map and \
           abs(self.timestamp_map[transaction_id] - time.time()) < self.window_size
CODE
Copy

```python
# Example code for a Bloom filter with rotating window
from bitarray import bitarray

class RotatingBloomFilter:
    def __init__(self, window_size, error_rate):
        self.window_size = window_size
        self.error_rate = error_rate
        self.filters = [bitarray() for _ in range(window_size)]
        self.current_filter_index = 0

    def add_transaction(self, transaction_id, timestamp):
        hash_value = hash(transaction_id) % self.window_size
        filter_index = (hash_value + self.current_filter_index * window_size) % window_size

        if not self.filters[filter_index].test(transaction_id):
            self.filters[filter_index].setall(True)

    def has_duplicate_in_window(self, transaction_id):
        for i in range(len(self.filters)):
            filter_index = (hash(transaction_id) + i * window_size) % len(self.filters)
            if self.filters[filter_index].test(transaction_id):
                return True
        return False

    def rotate_window(self):
        self.current_filter_index += 1
```

**Example input / output:**

```

Input:

SlidingWindowStore with window size of 30 seconds.
RotatingBloomFilter with error rate of 0.01 and window size of 30 seconds.

Output:

PYTHON
Copy
store = SlidingWindowStore(window_size=30)
bloom_filter = RotatingBloomFilter(window_size=30, error_rate=0.01)

# Adding transactions to the store and bloom filter
for i in range(len(transactionIds)):
    transaction_id = transactionIds[i]
    timestamp = timestamps[i]
    store.add_transaction(transaction_id, timestamp)
    bloom_filter.add_transaction(transaction_id, timestamp)

# Checking for duplicates
print(store.has_duplicate_in_window("tx1"))  # True if duplicate found within window
print(bloom_filter.has_duplicate_in_window("tx2"))  # True if potential duplicate (considering false positives)
```

**Data:**

```json
{
  "windowSize": 30,
  "transactionIds": ["tx1", "tx2", "tx3"],
  "timestamps": [time.time() - 10, time.time() - 5, time.time()]
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Transaction arrives
6. Is within current window?
7. Mark as duplicate, remove oldentries
8. Check Bloom filter forpotential duplicates
9. Is potential duplicate (falsepositive)?
10. Insert into Bloom filter
11. Mark as duplicate

