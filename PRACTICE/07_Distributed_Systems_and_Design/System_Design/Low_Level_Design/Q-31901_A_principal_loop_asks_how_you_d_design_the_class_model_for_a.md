---
type: "practice"
difficulty: "Deep"
---

# A principal loop asks how you'd design the class model for a 'projection' that serves multiple client shapes (GraphQL, REST, gRPC) from one domain aggregate without letting each transport's concerns leak into the domain; where does shaping live and how do you avoid three divergent DTOs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ DTOs and Domain Models](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/DTOs_and_Domain_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this LLD scenario, we need to design a class model that allows us to project data from a single domain aggregate while ensuring it can be consumed by multiple client shapes (GraphQL, REST, gRPC) without introducing concerns related to these transports into the domain layer. This is crucial for maintaining clean separation of concerns and ensuring robust and maintainable code.

#### Triage

The question is essentially asking how we can abstract data projection logic from our domain model so that different transport layers can consume it in their preferred formats without polluting the core business logic. We need to avoid creating multiple Data Transfer Objects (DTOs) for each client shape, which would introduce redundancy and maintainability issues.

#### First-Principles Solution
Define a Core Domain Model: This will encapsulate our domain logic and entities.
Create a Shaping Layer: This layer will handle the conversion of domain models into formats suitable for different clients.
Utilize a Common Interface or DTO Base Class: To ensure consistency across client shapes while allowing them to specialize their output.
Step-by-Step Solution

Define the Core Domain Model

The core aggregate should focus on business logic and persistence concerns, with no direct dependencies on transport mechanisms.

Create a Shaping Layer

Introduce an abstract Shaper class or interface that defines how to project data into different client shapes.
Implement specific shapers for each client (e.g., GraphQLShaper, RestShaper, GrpcShaper).
These shapers will take the domain model as input and produce a shape-specific output.

Use Common DTO Base Class

Define a common base class or interface that can be shared among multiple clients.
Clients can extend this base to add specific fields or logic required by their shape.

#### Example
Classes

PYTHON
from abc import ABC, abstractmethod

# Core Domain Model
class Order:
    def __init__(self, id, customer_id, status):
        self.id = id
        self.customer_id = customer_id
        self.status = status

# Shaper Interface
class Shaper(ABC):
    @abstractmethod
    def shape(self, order: Order) -> dict:
        pass

# GraphQL Shaper
class GraphQLShaper(Shaper):
    def shape(self, order: Order) -> dict:
        return {
            "id": order.id,
            "customer_id": order.customer_id,
            "status": order.status.value  # Assuming status is an Enum
        }

# Rest Shaper
class RestShaper(Shaper):
    def shape(self, order: Order) -> dict:
        return {
            "orderId": order.id,
            "customerId": order.customer_id,
            "status": order.status.name  # Assuming status is an Enum
        }

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By separating the shaping logic from the domain model, we ensure that our core business logic remains clean and focused on its primary responsibilities. This design also allows us to easily introduce new client shapes without affecting the domain layer.

#### Common Mistakes

Failing to separate concerns properly, leading to a mix of business logic and transport-specific code in the domain model.
Not defining a common base class or interface for shared attributes, causing duplication across multiple DTOs.
Ignoring potential edge cases where clients might require additional fields not present in the core model.

#### Follow-Ups
Discuss scenarios like handling nested data, optimizing performance for different transport layers, and ensuring thread safety if shapers are accessed concurrently.

**Code:**

```python
# Core Domain Model
class Order:
    def __init__(self, id, customer_id, status):
        self.id = id
        self.customer_id = customer_id
        self.status = status

# Shaper Interface
from abc import ABC, abstractmethod

class Shaper(ABC):
    @abstractmethod
    def shape(self, order: Order) -> dict:
        pass

# GraphQL Shaper
class GraphQLShaper(Shaper):
    def shape(self, order: Order) -> dict:
        return {
            "id": order.id,
            "customer_id": order.customer_id,
            "status": order.status.value  # Assuming status is an Enum
        }

# Rest Shaper
class RestShaper(Shaper):
    def shape(self, order: Order) -> dict:
        return {
            "orderId": order.id,
            "customerId": order.customer_id,
            "status": order.status.name  # Assuming status is an Enum
        }
```

**Example input / output:**

```text
order_data = Order(id=1, customer_id="001", status="COMPLETED")

graphql_response = GraphQLShaper().shape(order_data)
rest_response = RestShaper().shape(order_data)

print(graphql_response)  # {'id': 1, 'customer_id': '001', 'status': 'COMPLETED'}
print(rest_response)     # {'orderId': 1, 'customerId': '001', 'status': 'COMPLETED'}
```

**Data:**

```python
# Example Domain Object
order = Order(id=1, customer_id="001", status="COMPLETED")

# Shaping Data for GraphQL Client
graphql_shape = GraphQLShaper().shape(order)
print(graphql_shape)  # Output: {'id': 1, 'customer_id': '001', 'status': 'COMPLETED'}

# Shaping Data for REST Client
rest_shape = RestShaper().shape(order)
print(rest_shape)  # Output: {'orderId': 1, 'customerId': '001', 'status': 'COMPLETED'}
```

