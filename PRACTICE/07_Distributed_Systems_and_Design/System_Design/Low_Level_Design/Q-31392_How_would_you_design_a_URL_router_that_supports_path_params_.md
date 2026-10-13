---
type: "practice"
difficulty: "Deep"
---

# How would you design a URL router that supports path params, wildcards, and route priority so that /users/me doesn't get swallowed by /users/:id, and matching stays O(path-segments)?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tenant Isolation in Repositories](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a low-level URL router that supports path parameters, wildcards, and route priorities, we need to ensure that specific routes like /users/me are not overshadowed by more generic ones such as /users/:id. The key challenge is maintaining efficient and context-aware routing while keeping the implementation robust and scalable. Here's a step-by-step breakdown of how this can be achieved:

Understand the Question: The router must correctly match URLs based on path segments, taking into account parameters, wildcards, and priority levels.

#### Assumptions

Paths are defined as strings with placeholders like :id for dynamic values and * for wildcard matching.
Route priorities determine which route is chosen in case of multiple matches.
We aim for O(path-segments) time complexity, meaning the performance should scale linearly with the length of the path.

#### Solution

Segment Parsing: Break down each URL into segments and compare them to the defined routes.
Placeholder Handling: Identify and handle dynamic placeholders (:id) in routes.
Wildcard Matching: Implement wildcard handling to match patterns like *.
Priority-Based Routing: Ensure that routes with higher priority are matched first.

First Principles:

Start by defining the core components of the router, focusing on parsing and matching logic.
Use a data structure to represent routes, including placeholders and priorities.
Develop an algorithm that efficiently matches URLs against these routes.

#### Implementation
Steps:

PYTHON
class Route:
    def __init__(self, path, priority):
        self.path = path  # Path with placeholders like :id or *
        self.priority = priority

    def match(self, url_segments):
        for i, (path_segment, url_segment) in enumerate(zip_longest(self.path.split('/'), url_segments)):
            if not path_segment:
                continue
            if path_segment.startswith(':') and path_segment[1:] != url_segment:  # Dynamic placeholder check
                return False
            elif path_segment == '*':  # Wildcard match
                break
        else:
            return True

class Router:
    def __init__(self):
        self.routes = []

    def add_route(self, route):
        self.routes.append(route)

    def find_best_match(self, url_segments):
        best_priority = -float('inf')
        matching_route = None
        for route in sorted(self.routes, key=lambda r: r.priority, reverse=True):  # Prioritize higher routes first
            if route.match(url_segments) and route.priority > best_priority:
                best_priority = route.priority
                matching_route = route
        return matching_route

router = Router()
router.add_route(Route('/users/:id', priority=2))
router.add_route(Route('/users/me', priority=3))  # Higher priority than the above
router.add_route(Route('/posts/*', priority=1))

print(router.find_best_match(['users', 'me']))  # Should return '/users/me'

#### Closing Summary
and Common Mistakes:

The solution involves parsing URLs, handling dynamic placeholders and wildcards, and applying route priorities.
Key mistakes to avoid include incorrect placeholder matching logic, ignoring wildcard patterns, or not properly sorting routes by priority.

