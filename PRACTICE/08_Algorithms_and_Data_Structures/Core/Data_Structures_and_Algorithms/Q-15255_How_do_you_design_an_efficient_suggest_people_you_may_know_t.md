---
type: "practice"
difficulty: "Deep"
---

# How do you design an efficient 'suggest people you may know' that ranks by mutual-connection count without recomputing the whole graph, and where do you precompute versus query live?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Set Intersection Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Set_Intersection_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient "suggest people you may know" feature that ranks users based on mutual connections, we need to balance between precomputation and real-time querying. The goal is to minimize the computational overhead while ensuring accurate and timely suggestions. This problem involves understanding graph theory, data structures, and optimization techniques.

Firstly, let's break down what the question is asking:

We need a mechanism that suggests users based on mutual connections.
The ranking should be by mutual connection count.
We want to avoid recomputing the entire graph from scratch every time a suggestion request is made.

We will explore both precomputation and query-time strategies, discussing their trade-offs in terms of time complexity, space complexity, and accuracy.

Precomputation vs Query-Time Computation
Precomputation Strategy

Precomputing mutual connection counts can be done by iterating over all pairs of users to count common friends. This approach involves:

Building a graph representation.
Iterating through the graph to calculate the number of mutual connections for each pair of users.

#### Pros

Faster queries since we avoid recomputing every time.

#### Cons
istent and accurate results during query times.

#### Cons

High initial computational cost, especially for large graphs.
Storage requirements can be significant if not managed efficiently.
Query-Time Computation

We could choose to only compute mutual connections when a suggestion is requested. This approach involves:

Storing the graph structure efficiently.
Computing mutual connection counts on-the-fly during query times.

#### Pros

Lower initial storage and computational overhead.
Flexibility in dynamic changes to the graph without recomputing everything.

#### Cons

Higher latency for queries due to computation at runtime.
Inconsistent results if not optimized properly.
Hybrid Approach

A practical solution combines both precomputation and query-time optimization. We can precompute mutual connections between a subset of users or use sampling techniques, then dynamically update the counts when necessary.

Step-by-Step Solution

Graph Representation:

Use an adjacency list to represent the graph.
PYTHON
# Example in Python using adjacency lists
class User:
    def __init__(self, user_id):
        self.user_id = user_id
        self.neighbors = set()

users = {}

def add_friend(user1_id, user2_id):
    if user1_id not in users:
        users[user1_id] = User(user1_id)
    if user2_id not in users:
        users[user2_id] = User(user2_id)
    users[user1_id].neighbors.add(user2_id)
    users[user2_id].neighbors.add(user1_id)

Precomputation:

Compute mutual connections for a subset of users or using sampling.
PYTHON
def precompute_mutual_connections():
    from itertools import combinations

    # Sample a fraction of user pairs to precompute
    sampled_pairs = list(combinations(users.keys(), 2))

    for pair in sampled_pairs:
        u1, u2 = pair
        mutual_friends_count = len(users[u1].neighbors.intersection(users[u2].neighbors))
        users[u1].mutual_friends_count[u2] = mutual_friends_count
        users[u2].mutual_friends_count[u1] = mutual_friends_count

    # Store the precomputed values in a separate structure or database

Query Handling:

For real-time queries, use the stored precomputation results and only update when necessary.
PYTHON
def suggest_users(user_id):
    recommended_users = []

    for other_user in users:
        if other_user != user_id:
            mutual_friends_count = 0

            # Use precomputed values if available
            if user_id in users[other_user].mutual_friends_count:
                mutual_friends_count = users[other_user].mutual_friends_count[user_id]

            # If not, compute dynamically (e.g., on a small sample of friends)
            if mutual_friends_count == 0 and len(users[user_id].neighbors) < SAMPLE_SIZE:
                for friend in users[user_id].neighbors:
                    mutual_friends_count += len(users[friend].neighbors.intersection(users[other_user].neighbors))

            # Add to recommendations
            recommended_users.append((other_user, mutual_friends_count))

    # Sort by mutual friends count and return top N suggestions
    recommended_users.sort(key=lambda x: -x[1])
    return [user[0] for user in recommended_users[:N]]

#### Closing Summary

By combining precomputation with real-time updates, we can achieve a balance between computational efficiency and accuracy. The key is to carefully manage the trade-offs based on the size of the graph and the frequency of updates.

#### Common Mistakes/Follow-Ups
Overlooking the initial cost of precomputation.
Ignoring the potential for inaccurate results if not using sampling properly.
Failing to consider edge cases, such as users with no mutual friends or dynamically changing graphs.
Not optimizing storage and query times, leading to performance bottlenecks.

**Comparison:**

| User | Friends |
|------|---------|
| 1    | 2, 3    |
| 2    | 1, 4    |
| 3    | 1, 5    |
| 4    | 2, 6    |
| 5    | 3, 7    |
| 6    | 4       |
| 7    | 5       |

**Code:**

```python
# Graph class for user connections
class User:
    def __init__(self, user_id):
        self.user_id = user_id
        self.neighbors = set()

users = {}

def add_friend(user1_id, user2_id):
    if user1_id not in users:
        users[user1_id] = User(user1_id)
    if user2_id not in users:
        users[user2_id] = User(user2_id)
    users[user1_id].neighbors.add(user2_id)
    users[user2_id].neighbors.add(user1_id)

def precompute_mutual_connections():
    from itertools import combinations

    # Sample a fraction of user pairs to precompute
    sampled_pairs = list(combinations(users.keys(), 2))

    for pair in sampled_pairs:
        u1, u2 = pair
        mutual_friends_count = len(users[u1].neighbors.intersection(users[u2].neighbors))
        users[u1].mutual_friends_count[u2] = mutual_friends_count
        users[u2].mutual_friends_count[u1] = mutual_friends_count

def suggest_users(user_id):
    recommended_users = []

    for other_user in users:
        if other_user != user_id:
            mutual_friends_count = 0

            # Use precomputed values if available
            if user_id in users[other_user].mutual_friends_count:
                mutual_friends_count = users[other_user].mutual_friends_count[user_id]

            # If not, compute dynamically (e.g., on a small sample of friends)
            if mutual_friends_count == 0 and len(users[user_id].neighbors) < SAMPLE_SIZE:
                for friend in users[user_id].neighbors:
                    mutual_friends_count += len(users[friend].neighbors.intersection(users[other_user].neighbors))

            # Add to recommendations
            recommended_users.append((other_user, mutual_friends_count))

    # Sort by mutual friends count and return top N suggestions
    recommended_users.sort(key=lambda x: -x[1])
    return [user[0] for user in recommended_users[:N]]
```

**Example input / output:**

```text
# Example input and output

# Adding friends
add_friend(1, 2)
add_friend(1, 3)
add_friend(2, 4)
add_friend(3, 5)
add_friend(4, 6)
add_friend(5, 7)

# Precompute mutual connections (simplified for example)
precompute_mutual_connections()

# Suggest users for user 1
suggest_users(1)  # Output: [2, 3, 4, 5]
```

**Data:**

```json
{
  "users": [
    {"id": 1, "friends": [2, 3]},
    {"id": 2, "friends": [1, 4]},
    {"id": 3, "friends": [1, 5]},
    {"id": 4, "friends": [2, 6]},
    {"id": 5, "friends": [3, 7]},
    {"id": 6, "friends": [4]},
    {"id": 7, "friends": [5]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Adds Friend
4. Add to Graph
5. Is Precomputation Needed?
6. Precompute MutualConnections
7. Handle Query withPrecomputed Data or DynamicCalculation
8. Return Top NRecommendations

