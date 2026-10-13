---
type: "practice"
difficulty: "Expert"
---

# How would you design an efficient membership-and-count structure for 'how many users have seen this notification' at massive scale while staying within a tight per-notification memory budget?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Set Intersection Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Set_Intersection_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To efficiently manage "how many users have seen this notification" with minimal memory overhead, we need to design a scalable and space-efficient data structure. The core challenge is to accurately count unique user IDs while staying within strict memory limits per notification. This problem requires balancing trade-offs between accuracy, memory usage, and performance.

#### Triage
Key Elements

The question asks us to design an efficient membership-and-count structure under tight constraints. Specifically:

Memory Budget: We must ensure that the solution does not exceed a certain memory limit per notification.
Counting Unique Users: We need to accurately track how many distinct users have seen each notification.

#### First-Principles Solution

To address this, we can use a combination of hash sets and probabilistic data structures like Bloom Filters. Here's a detailed breakdown:

Step 1: Understanding the Trade-offs

A straightforward approach using a hash set would use O(n) memory for n unique users, which is impractical given tight constraints. Instead, we'll leverage a Bloom Filter to reduce memory usage at the cost of some accuracy.

Step 2: Choosing the Right Data Structure

Bloom Filters: These are space-efficient probabilistic data structures that can test whether an element is a member of a set with high probability but may produce false positives.

Step 3: Designing the Bloom Filter
Hash Functions: Use multiple independent hash functions to map user IDs into bits in the Bloom Filter.
False Positive Rate: Set the size and number of hash functions such that the false positive rate is acceptable (e.g., less than 0.1%).
Memory Efficiency: Ensure the filter fits within the allowed memory budget per notification.
Step 4: Integrating with Unique Count

To maintain an accurate count, we can use a combination Bloom Filter and a separate counter:

Bloom Filter: Tracks if a user has seen the notification.
Counter: Keeps track of the number of unique users who have seen the notification.

#### Implementation
Details

Here's a simplified implementation outline:

PYTHON
import hashlib

class MembershipCount:
    def __init__(self, max_users=100000, false_positive_rate=0.001):
        self.max_users = max_users
        self.bloom_filter_size = int(- (max_users * math.log(false_positive_rate)) / (math.log(2)**2))
        self.num_hash_functions = int(self.bloom_filter_size / max_users)
        self.bit_array = [False] * self.bloom_filter_size
        self.unique_count = 0

    def add_user(self, user_id):
        if not self.is_seen(user_id):
            for i in range(self.num_hash_functions):
                idx = self._hash_function(user_id, i) % self.bloom_filter_size
                self.bit_array[idx] = True
            self.unique_count += 1

    def is_seen(self, user_id):
        for i in range(self.num_hash_functions):
            idx = self._hash_function(user_id, i) % self.bloom_filter_size
            if not self.bit_array[idx]:
                return False
        return True

    def _hash_function(self, user_id, i):
        return int(hashlib.sha256(f"{user_id}{i}".encode()).hexdigest(), 16)

# Example Usage:
notification = MembershipCount(max_users=10000, false_positive_rate=0.001)
users = [1, 2, 3, 4, 5]
for user in users:
    notification.add_user(user)
print(notification.unique_count)  # Output: 5

#### Closing Summary
+ Common Mistakes/Follow-ups

This solution efficiently tracks unique users while keeping memory usage low. The Bloom Filter ensures minimal memory overhead and allows for near-constant time operations. However, it trades off some accuracy with false positives.

#### Common Mistakes
Overlooking False Positives: Not understanding the implications of false positives in critical applications.
Ignoring Memory Budgets: Failing to account for strict memory limits per notification.
Simplistic Hash Functions: Using overly simplistic or non-cryptographic hash functions, which can lead to more collisions.

#### Follow-ups
How would you handle updates and deletions?
What if the false positive rate needs to be extremely low (e.g., 0.01%)?
Can you optimize further for memory usage at the cost of slower operations?

**Code:**

```python
import math

class MembershipCount:
    def __init__(self, max_users=100000, false_positive_rate=0.001):
        self.max_users = max_users
        self.bloom_filter_size = int(- (max_users * math.log(false_positive_rate)) / (math.log(2)**2))
        self.num_hash_functions = int(self.bloom_filter_size / max_users)
        self.bit_array = [False] * self.bloom_filter_size
        self.unique_count = 0

    def add_user(self, user_id):
        if not self.is_seen(user_id):
            for i in range(self.num_hash_functions):
                idx = self._hash_function(user_id, i) % self.bloom_filter_size
                self.bit_array[idx] = True
            self.unique_count += 1

    def is_seen(self, user_id):
        for i in range(self.num_hash_functions):
            idx = self._hash_function(user_id, i) % self.bloom_filter_size
            if not self.bit_array[idx]:
                return False
        return True

    def _hash_function(self, user_id, i):
        return int(hashlib.sha256(f"{user_id}{i}".encode()).hexdigest(), 16)
```

**Example input / output:**

```text
# Example Input/Output
notification = MembershipCount(max_users=10000, false_positive_rate=0.001)
users = [1, 2, 3, 4, 5]
for user in users:
    notification.add_user(user)
print(notification.unique_count)  # Output: 5
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. User Seen Notification
6. Is User Added to Bloom Filter?
7. Increment Unique Count
8. Bloom Filter Updated
9. Check if Already in BloomFilter
10. False Positive, Do Nothing
11. Add User to Bloom Filter andIncrement Unique Count

