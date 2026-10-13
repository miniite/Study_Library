---
type: "practice"
difficulty: "Deep"
---

# Design a memory-bounded structure holding 'the k most recent distinct users' per resource; why is naive dedup-plus-truncate wrong, and what ordering must you maintain to stay correct under updates?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Exactly Once Dedup Structures](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Exactly_Once_Dedup_Structures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: To design a memory-bounded structure holding the k most recent distinct users per resource, we need to implement a solution that supports efficient insertion and lookup while maintaining a specific order. The naive approach of simply deduplicating and truncating is incorrect because it fails to handle updates properly. We must maintain an ordered collection such as a doubly linked list or a balanced tree to ensure correctness.

#### Triaging the Question
The core of this question revolves around implementing an efficient data structure that can hold the k most recent distinct users for each resource while being memory bounded. The interviewer is probing your understanding of data structures, algorithms, and their trade-offs in terms of performance and complexity. Key assumptions include:

Memory constraints are a hard limit.
Users and resources are uniquely identifiable.
Operations (insertions and lookups) must be efficient.

#### First Principles Solution

Data Structure Choice: We need to choose a data structure that supports fast insertion, deletion, and lookup while maintaining an ordered sequence of elements.
Handling Duplicates: To avoid storing the same user multiple times under the same resource, we can use a hash map (dictionary) to track which users are already present for each resource.
Maintaining Order: A doubly linked list or a balanced tree like AVL/BST is necessary to maintain order and enable efficient operations.

Detailed Steps:

Hash Map for User Tracking: Use a hash map where the key is the user ID/resource combination, and the value points to an element in our ordered collection.
Ordered Collection: Implement either a doubly linked list or balanced tree (e.g., AVL Tree) to maintain order. Each node stores the user ID and optionally the resource ID if needed for clarity.
Insertion Process:
Check the hash map to see if the user is already present for the given resource.
If not, insert the user in the ordered collection while maintaining its order (e.g., by timestamp).
Update the hash map with the new node reference.
Deletion Process:
Remove the user from the ordered collection based on the insertion order.
Update the hash map to reflect this change.

#### Example
Implementation and Dry-Run: Consider a scenario where we have two resources, A and B, and users are inserted over time:

PYTHON
class ResourceUserTracker:
    def __init__(self, k):
        self.k = k
        self.users_by_resource = {}  # Map from resource to ordered collection of users
        self.current_user_id = 0

    class UserNode:
        def __init__(self, user_id, resource):
            self.user_id = user_id
            self.resource = resource
            self.prev = None
            self.next = None

    def add_user(self, user_id, resource):
        if (user_id, resource) in self.users_by_resource:
            return False  # User already exists for this resource

        new_node = self.UserNode(user_id, resource)

        # Insert node into the ordered collection
        tail = self.users_by_resource[resource].tail
        prev_tail = tail.prev
        if not tail or (new_node.user_id < tail.user_id):
            new_node.next = tail
            tail.prev = new_node
            if prev_tail:
                prev_tail.next = new_node
            else:
                self.users_by_resource[resource].head = new_node
            new_node.prev = prev_tail
        else:
            # Find the correct position to insert
            current = self.users_by_resource[resource].head
            while current and (new_node.user_id > current.user_id):
                current = current.next
            new_node.next = current
            current.prev = new_node
            if current.prev:
                current.prev.next = new_node
            else:
                self.users_by_resource[resource].head = new_node
            new_node.prev = current.prev

        # If the collection exceeds capacity, remove the last element
        while len(self.users_by_resource[resource]) > self.k:
            del self.users_by_resource[resource][self.users_by_resource[resource][-1]]

        return True

# Example usage
tracker = ResourceUserTracker(3)
tracker.add_user(101, 'A')  # Adds user 101 for resource A
tracker.add_user(202, 'B')  # Adds user 202 for resource B
tracker.add_user(101, 'A')  # Does not add as user 101 already exists for resource A
tracker.add_user(303, 'A')  # Replaces user 101 with user 303 in the list

# The final state of the tracker would be:
# - Resource A: [303]
# - Resource B: [202]

#### Closing Summary
and Common Mistakes/Follow-Ups: This approach ensures that we maintain the correct order and handle duplicates efficiently. The key is to use a combination of hash maps for quick lookups and an ordered collection like a doubly linked list or balanced tree for maintaining sequence.

A common mistake in this implementation is failing to correctly manage the removal process, especially when the collection exceeds its capacity. Always ensure that nodes are properly removed from both ends of the list/tree as needed.

Common follow-ups could involve discussing:

Performance trade-offs between different data structures.
How to scale this solution for a large number of users and resources.
Handling concurrent modifications in a multi-threaded environment.

**Example input / output:**

```

Example input -> output:

PLAINTEXT
Copy
Input 1: tracker.add_user(101, 'A')
Output 1: True

Input 2: tracker.add_user(202, 'B')
Output 2: True

Input 3: tracker.add_user(101, 'A')
Output 3: False

Input 4: tracker.add_user(303, 'A')
Output 4: True
```

