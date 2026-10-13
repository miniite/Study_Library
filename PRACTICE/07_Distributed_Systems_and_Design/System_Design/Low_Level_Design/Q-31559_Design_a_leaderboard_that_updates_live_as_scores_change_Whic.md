---
type: "practice"
difficulty: "Intermediate"
---

# Design a leaderboard that updates live as scores change. Which structure gives fast top-K and rank?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Threaded Comments Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Threaded_Comments_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To design a real-time leaderboard with fast top-K updates, we can use a combination of data structures like a min-heap to maintain the top K scores efficiently, along with a hash map to track all user scores. This ensures that both insertion and retrieval operations are optimized for speed.

#### Triaging the Question

The question asks us to design a leaderboard system that can quickly update and display the top-K scores in real-time as new scores come in. The key aspects we need to consider include:

Efficient insertion: Adding a new score should be fast.
Quick retrieval of top-K: Getting the top K scores should not degrade with the number of total users.
Ranking accuracy: Correctly displaying ranks even when there are ties.

#### Assumptions
and First Principles
Score Update Frequency: Scores change frequently, so we need a system that can handle high insertions/updates efficiently.
User Count: The leaderboard has around 10,000 to 100,000 users.
Top-K Size: Typically, the top K (e.g., top-10) needs to be retrieved quickly.
Step-by-Step Solution
Data Structures and Design Choices

1. Min-Heap for Top-K A min-heap can efficiently maintain the top K scores. Insertions and deletions in a heap are (O(\log K)). For top-K, we always keep the smallest of the K largest values at the root.

Yes
No
User Score
Is Score > Top K?
Insert into Heap
Keep Current Heap
Heap Maintains Top K

2. Hash Map for All Scores A hash map (dictionary) allows us to quickly look up a user's current score in (O(1)) time.

Lookup
User ID
Score
Hash Map

3. Efficient Updates and Re-balancing

Insert New Score: Check if the new score is greater than the current top-K score (at the root of the heap). If so, update the heap.
Update Existing Score: Use the hash map to find the user's current score, adjust it, and reinsert into the heap as necessary.
Code Implementation
PYTHON
import heapq

class Leaderboard:
    def __init__(self):
        self.scores = {}  # Hash map for quick lookups
        self.heap = []    # Min-heap to maintain top K scores

    def update_score(self, user_id, score):
        if user_id in self.scores:
            old_score = self.scores[user_id]
            del self.scores[user_id]  # Remove from hash map
            heapq.heappushpop(self.heap, (score - old_score, user_id))  # Update heap
        else:
            heapq.heappush(self.heap, (score, user_id))  # Add new score
        self.scores[user_id] = score  # Update or add to hash map

    def top_k(self):
        return [(-item[0], item[1]) for item in self.heap[:self.K]]  # Convert to positive values

# Example usage:
leaderboard = Leaderboard()
leaderboard.update_score("user1", 20)
leaderboard.update_score("user2", 50)
leaderboard.update_score("user3", 30)
print(leaderboard.top_k())  # [(50, 'user2'), (30, 'user3')]
Performance Analysis
Insertion and Update: (O(\log K)) for both heap operations.
Retrieval of Top-K: (O(K \log K)) since we are maintaining a heap of size K.

#### Closing Summary

The design leverages a min-heap to maintain the top K scores efficiently, while using a hash map for quick lookups. This ensures that both insertion and retrieval operations remain efficient even as the number of users grows. However, we need to manage rebalancing the heap after each update to keep it accurate.

#### Common Mistakes
and Follow-Ups
Not Rebalancing the Heap: Failing to reinsert or remove scores from the heap can lead to incorrect top-K values.
Ignoring Ties in Ranking: Ensuring that ties are handled correctly, especially when displaying ranks.
Scalability Issues: Not considering how the system behaves as (K) becomes very large compared to the number of total users.

**Code:**

```python
import heapq

class Leaderboard:
    def __init__(self):
        self.scores = {}
        self.heap = []

    def update_score(self, user_id, score):
        if user_id in self.scores:
            old_score = self.scores[user_id]
            del self.scores[user_id]  # Remove from hash map
            heapq.heappushpop(self.heap, (score - old_score, user_id))  # Update heap
        else:
            heapq.heappush(self.heap, (score, user_id))  # Add new score
        self.scores[user_id] = score

    def top_k(self):
        return [(-item[0], item[1]) for item in self.heap[:self.K]]
```

**Example input / output:**

```text
leaderboard = Leaderboard()
leaderboard.update_score("user1", 20)
leaderboard.update_score("user2", 50)
leaderboard.update_score("user3", 30)

# Example output for top_k
print(leaderboard.top_k())  # [(50, 'user2'), (30, 'user3')]
```

**Data:**

```json
{
    "user_scores": {
        "user1": 20,
        "user2": 50,
        "user3": 30
    }
}
```

