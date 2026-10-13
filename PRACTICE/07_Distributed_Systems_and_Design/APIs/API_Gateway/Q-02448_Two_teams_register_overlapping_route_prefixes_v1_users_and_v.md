---
type: "practice"
difficulty: "Intermediate"
---

# Two teams register overlapping route prefixes (/v1/users and /v1/users/*) on the same gateway and requests match the wrong upstream; how do you define deterministic precedence rules and detect conflicts at config-load time?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Control Plane and Data Plane Design](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Routing_and_BFF_Patterns/Control_Plane_and_Data_Plane_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to address a common issue in API Gateway configurations where two teams register overlapping route prefixes that can lead to unexpected behavior. The goal is to define deterministic precedence rules and detect conflicts at configuration load time. This ensures that the API Gateway behaves predictably and avoids routing errors.

The question requires us to understand how to prioritize routes, handle overlapping paths, and implement conflict detection mechanisms effectively. We will start by defining a clear set of rules for route precedence, ensuring that no conflicting configurations can be loaded into the gateway.

#### Triaging the Question
What is being asked: Define deterministic precedence rules and detect conflicts at config-load time.
What the interviewer is probing:
Understanding of routing principles in API Gateways.
Ability to define a clear hierarchy for route matching.
Knowledge on how to handle overlapping paths and ensure no ambiguity in routing decisions.

#### Assumptions
We are working with an HTTP-based API Gateway.
The gateway uses URL path patterns for routing.
Both teams have defined their routes using standard HTTP methods (e.g., GET, POST).
Step-by-Step Solution from First Principles

Route Definition and Precedence Rules:

Define a set of rules that determine which route takes precedence over others when there are overlapping paths. For example:
Routes with more specific path patterns should take precedence.
Routes for the same path pattern but different HTTP methods should be ordered based on their intended usage (e.g., POST before GET).

Conflict Detection:

Implement a mechanism to detect and prevent conflicting routes from being loaded simultaneously. This can involve:
Validating route paths during configuration load.
Ensuring that no two routes have the same path with different HTTP methods or overlapping specificity.

#### Example Scenario

Team A defines /v1/users (GET, POST).
Team B defines /v1/users/* (PUT, DELETE).

Precedence Rules:

/v1/users/* should take precedence over /v1/users because it is more specific.
Within the same path, different HTTP methods should be ordered by their typical usage:
POST before GET
PUT before DELETE

Conflict Detection Mechanism:

During configuration load time, validate that no two routes conflict with each other.
A simple algorithm to detect conflicts could involve:
Parsing all route definitions and storing them in a data structure (e.g., a list or map).
For each route, check if there is another route with the same path but different method.
If such a conflict is detected, raise an error during configuration load.
Detailed Implementation

Here’s a sample implementation in Python:

PYTHON
class Route:
    def __init__(self, prefix, http_method):
        self.prefix = prefix
        self.http_method = http_method

def validate_routes(routes):
    # Map to store prefixes and their corresponding methods
    route_map = {}

    for route in routes:
        if route.prefix not in route_map:
            route_map[route.prefix] = [route.http_method]
        else:
            existing_methods = route_map[route.prefix]
            if route.http_method in existing_methods:
                raise ValueError(f"Conflict detected: {route.prefix} with method {route.http_method}")
            else:
                # Maintain order of methods based on their typical usage
                insert_index = next((i for i, m in enumerate(existing_methods) if m > route.http_method), len(existing_methods))
                existing_methods.insert(insert_index, route.http_method)

def main():
    routes = [
        Route("/v1/users", "GET"),
        Route("/v1/users", "POST"),
        Route("/v1/users/*", "PUT"),  # This should be validated
        Route("/v1/users/*", "DELETE")  # This should be validated
    ]

    try:
        validate_routes(routes)
        print("No conflicts detected.")
    except ValueError as e:
        print(e)

if __name__ == "__main__":
    main()

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary

Define clear precedence rules for route matching.
Implement conflict detection mechanisms to prevent overlapping paths.
Use a data structure like a map to store and validate routes during configuration load.

#### Common Mistakes

Failing to define specific order rules (e.g., POST before GET).
Not implementing proper validation at config-load time, leading to runtime issues.
Ignoring the importance of HTTP method ordering in conflict detection.

#### Follow-Ups

How would you handle route updates and rollbacks?
What about third-party services using these routes?
How do you ensure that your solution is scalable with a large number of routes?

**Code:**

```python
class Route:
    def __init__(self, prefix, http_method):
        self.prefix = prefix
        self.http_method = http_method

def validate_routes(routes):
    route_map = {}

    for route in routes:
        if route.prefix not in route_map:
            route_map[route.prefix] = [route.http_method]
        else:
            existing_methods = route_map[route.prefix]
            if route.http_method in existing_methods:
                raise ValueError(f"Conflict detected: {route.prefix} with method {route.http_method}")
            else:
                insert_index = next((i for i, m in enumerate(existing_methods) if m > route.http_method), len(existing_methods))
                existing_methods.insert(insert_index, route.http_method)

def main():
    routes = [
        Route("/v1/users", "GET"),
        Route("/v1/users", "POST"),
        Route("/v1/users/*", "PUT"),  # This should be validated
        Route("/v1/users/*", "DELETE")  # This should be validated
    ]

    try:
        validate_routes(routes)
        print("No conflicts detected.")
    except ValueError as e:
        print(e)

if __name__ == "__main__":
    main()
```

