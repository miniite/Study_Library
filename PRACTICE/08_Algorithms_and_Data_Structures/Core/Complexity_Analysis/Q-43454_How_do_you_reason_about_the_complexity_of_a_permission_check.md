---
type: "practice"
difficulty: "Deep"
---

# How do you reason about the complexity of a permission check that walks a role/group hierarchy (tree/DAG) per request, and how you'd flatten or cache it to avoid repeated O(depth) traversals?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Complexity vs Real Performance](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Complexity_vs_Real_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reason about the complexity of a permission check in a role/group hierarchy, we need to understand both time and space complexities involved in walking through the hierarchy per request and how to optimize it using caching or flattening techniques.

#### Triage

The problem revolves around ensuring that each user's permissions are correctly evaluated based on their roles and groups, while minimizing computational overhead. The key is to recognize that a naive approach might result in an O(depth) traversal for every permission check, where "depth" refers to the length of the longest path from the root role to the specific role or group we're querying.

#### First-Principles Solution
Step 1: Analyze the Problem

The hierarchy can be modeled as a tree (if hierarchical) or directed acyclic graph (DAG) if there are cycles. For each permission request, we need to check all roles and groups that a user belongs to. If not optimized, this could lead to repeated traversals of the same path.

Step 2: Time Complexity
Naive Approach: Every time a permission is checked, the entire hierarchy from the top (root) to the specific role or group must be traversed.
Optimization Needed: To avoid this O(depth) traversal every time, we need to cache the results of previous checks.
Step 3: Space Complexity
Caching: We need a data structure to store intermediate results. A simple approach is using a hash map where keys are user IDs and values are sets of roles or groups they belong to.
Flattening: Another technique is to precompute all possible paths (flattened hierarchy) for each role or group.
Step 4: Implementation

Caching Approach:

PYTHON
class PermissionCache:
    def __init__(self):
        self.cache = {}

    def get_permissions(self, user_id):
        if user_id in self.cache:
            return self.cache[user_id]
        # Simulate fetching permissions from database or other sources
        roles_and_groups = fetch_user_roles(user_id)
        self.cache[user_id] = roles_and_groups
        return roles_and_groups

def check_permission(user_id, required_role):
    cache = PermissionCache()
    user_permissions = cache.get_permissions(user_id)
    return required_role in user_permissions

Flattening Approach:

PYTHON
class RoleHierarchy:
    def __init__(self):
        self.hierarchy = {}  # Node -> [Children]

    def build_hierarchy(self, root_id):
        for node_id in self.hierarchy[root_id]:
            self.build_hierarchy(node_id)

    def check_permission(self, user_id, required_role):
        flattened_user_roles = set()
        stack = [(user_id, [])]
        while stack:
            current_id, path = stack.pop()
            if current_id in self.hierarchy:
                for child_id in self.hierarchy[current_id]:
                    new_path = path + [child_id]
                    stack.append((child_id, new_path))
            elif current_id in flattened_user_roles or required_role in path:
                return True
        return False

def build_hierarchy_tree(root_id):
    hierarchy = RoleHierarchy()
    hierarchy.build_hierarchy(root_id)
    return hierarchy

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the key to optimizing permission checks is avoiding repeated traversals. Caching and flattening are effective techniques:

Caching: Store intermediate results to avoid recomputing.
Flattening: Precompute all paths to quickly check permissions.

#### Common mistakes
include:

Not recognizing the need for optimization in recursive or iterative processes.
Overlooking edge cases, such as roles that can be inherited from multiple parents.
Not considering memory usage if caching is employed extensively.
Common Follow-Ups
How would you implement a more complex hierarchy with overlapping roles?
What about scenarios where permissions are updated frequently?
How do you handle circular references in the role hierarchy?

**Code:**

```

class PermissionCache: def init(self): self.cache = {}

CODE
Copy
def get_permissions(self, user_id):
    if user_id in self.cache:
        return self.cache[user_id]
    roles_and_groups = fetch_user_roles(user_id)
    self.cache[user_id] = roles_and_groups
    return roles_and_groups

def check_permission(user_id, required_role): cache = PermissionCache() user_permissions = cache.get_permissions(user_id) return required_role in user_permissions

CODE
Copy

```python
class RoleHierarchy:
    def __init__(self):
        self.hierarchy = {}

    def build_hierarchy(self, root_id):
        for node_id in self.hierarchy[root_id]:
            self.build_hierarchy(node_id)

    def check_permission(self, user_id, required_role):
        flattened_user_roles = set()
        stack = [(user_id, [])]
        while stack:
            current_id, path = stack.pop()
            if current_id in self.hierarchy:
                for child_id in self.hierarchy[current_id]:
                    new_path = path + [child_id]
                    stack.append((child_id, new_path))
            elif current_id in flattened_user_roles or required_role in path:
                return True
        return False

def build_hierarchy_tree(root_id):
    hierarchy = RoleHierarchy()
    hierarchy.build_hierarchy(root_id)
    return hierarchy
```

**Data:**

```json
{
  "hierarchy": {
    "root": ["admin", "user"],
    "admin": ["moderator"],
    "moderator": []
  }
}
```

