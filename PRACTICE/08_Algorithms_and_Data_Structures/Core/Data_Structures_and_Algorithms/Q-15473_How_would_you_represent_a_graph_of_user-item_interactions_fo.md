---
type: "practice"
difficulty: "Intermediate"
---

# How would you represent a graph of user-item interactions for a recommender, and when is an adjacency list better than a matrix?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Shortest Paths](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Shortest_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a system to represent user-item interactions in a recommender, we need to balance between memory usage, speed of operations like fetching neighbors or finding edge weights, and ease of updates. An adjacency list is often preferable over an adjacency matrix when the graph is sparse because it uses less space and is more efficient for certain types of queries.

#### Triage

The question asks us to compare two common ways of representing a user-item interaction graph: an adjacency list and an adjacency matrix. The core concept here is understanding the trade-offs between these representations in terms of space complexity, time complexity, and ease of updates.

#### First-Principles Solution
Adjacency List Representation

An adjacency list consists of a list of nodes (or vertices) where each node contains a list of all its neighbors. This representation is particularly useful for sparse graphs, which are common in recommender systems because the number of interactions between users and items can be significantly less than ( n \times m ), where ( n ) is the number of users and ( m ) is the number of items.

Adjacency Matrix Representation

An adjacency matrix uses a 2D array to represent the graph. Each row and column corresponds to a node, and the value at position (i, j) indicates whether there is an edge from node i to node j. For dense graphs or when every user interacts with almost all items, this representation can be more efficient.

When Adjacency List is Better
Memory Efficiency: In sparse graphs, adjacency lists use less memory because they only store the actual interactions between users and items.
Efficient Retrieval of Neighbors: Operations like finding a list of neighbors for a given user are faster with an adjacency list compared to a matrix. You can simply retrieve the entire list from the corresponding node in the adjacency list, whereas you would need to check every row in the matrix.
Ease of Update: Adding or removing interactions is more straightforward in an adjacency list. For example, if a new interaction occurs between user ( u ) and item ( i ), we can simply add this pair to the corresponding adjacency lists.
When Adjacency Matrix is Better
Symmetry Handling: If there are symmetric relationships (e.g., bidirectional interactions like "user A likes item B" and "item B is liked by user A"), an adjacency matrix naturally handles these.
Efficient Edge Existence Check: Checking if an edge exists between two nodes in a matrix involves just one lookup, whereas with an adjacency list, you might need to traverse a linked list.

#### Closing Summary

In summary, for sparse interaction graphs like those found in recommenders, adjacency lists are generally more space-efficient and provide faster neighbor retrieval. However, they may not be as straightforward for operations that benefit from the symmetry of relationships or require quick edge existence checks.

#### Common Mistakes/Follow-Ups
Misunderstanding Density: Often, interviewees confuse sparse graphs with dense ones. A graph can have many interactions but still be sparse if ( n \times m ) is much larger than the number of actual interactions.
Ignoring Symmetry Requirements: For systems where bidirectional relationships are crucial (e.g., friendship in social networks), an adjacency matrix might be preferred despite higher space requirements.
Not Considering Graph Operations: Different operations have different optimal representations. For instance, if you frequently need to find all users interacting with a given item, an adjacency list could still be more efficient.

**Comparison:**

```
User ID	Neighbors
user101	[502]
user102	[503, 504]
```

**Code:**

```python
# Example of Adjacency List Representation in Python

class UserItemGraph:
    def __init__(self):
        self.users = {}

    def add_user_item_interaction(self, user_id, item_id):
        if user_id not in self.users:
            self.users[user_id] = []
        self.users[user_id].append(item_id)

    def get_neighbors_for_user(self, user_id):
        return self.users.get(user_id, [])
```

**Example input / output:**

```text
Given the input:
- Add user-item interactions: `add_user_item_interaction('user101', 502)`, `add_user_item_interaction('user102', 503)`, and `add_user_item_interaction('user102', 504)`.

 The output for neighbors of 'user102' would be:
- `get_neighbors_for_user('user102')`: `[503, 504]`
```

**Data:**

```json
{
  "users": {
    "user101": [502],
    "user102": [503, 504],
    "user103": []
  }
}
```

**Flow:**

1. Yes
2. No
3. User Item Interaction
4. Add User-Item Interaction
5. Does the User Exist?
6. Append Item to User's List
7. Create New Entry for User withItem
8. Get Neighbors for User
9. List of User's Interactions

