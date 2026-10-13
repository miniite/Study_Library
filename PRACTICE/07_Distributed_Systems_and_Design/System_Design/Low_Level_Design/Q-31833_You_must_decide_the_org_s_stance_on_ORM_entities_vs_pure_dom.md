---
type: "practice"
difficulty: "Deep"
---

# You must decide the org's stance on ORM entities vs pure domain objects: should the DB shape drive the domain model or vice versa? Explain the long-term cost of each at scale and how you'd guide teams to keep persistence concerns from bleeding into the domain.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ DTOs and Domain Models](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/DTOs_and_Domain_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between ORM entities vs pure domain objects, it's crucial to understand that both approaches have their trade-offs. ORM (Object-Relational Mapping) entities are shaped by the database schema, whereas pure domain objects reflect business logic and rules. At scale, each approach has long-term costs and benefits that need careful consideration.

The primary question is whether the database should dictate the shape of your domain model or if you should design the domain model independently and let it drive the persistence layer. In this deep dive, we'll explore both perspectives, their implications, and provide guidance on how to maintain a clean separation between persistence concerns and the core business logic.

First, let's clarify what each approach entails:

ORM Entities: These entities map directly to database tables, with properties corresponding to columns and relationships mirroring foreign key relationships. This can lead to tight coupling between your application code and the database schema.
Pure Domain Objects: These are focused on representing business concepts and rules, without direct knowledge of storage mechanisms like databases. They encapsulate the logic that defines how entities interact within the domain.

Now, let's break down the long-term costs and benefits of each approach at scale:

ORM Entities

Benefits:

Simplicity: Direct mapping makes it easier to start projects where database schema is clear and straightforward.

#### Cons
istency: Ensures that your application can take full advantage of database features like indexes, constraints, and transactions.

Costs:

Tight Coupling: Changes in the database schema require changes in the codebase. This can lead to brittleness if not managed carefully.
Maintenance Overhead: Frequent schema changes can increase maintenance effort as the application evolves.
Complexity: For large and complex systems, managing entity relationships can become cumbersome.
Pure Domain Objects

Benefits:

Loose Coupling: The domain model is decoupled from storage mechanisms, allowing for flexible and independent evolution of both.
Maintainability: Easier to reason about business logic without the noise of database-specific concerns.
Testability: Pure objects are easier to test in isolation.

Costs:

Complexity: Requires more effort to implement persistence concerns separately. You might need additional tools or patterns like repositories, mappers, or DTOs (Data Transfer Objects).

#### Performance Considerations
Without ORM, you lose the convenience of automatic mapping and querying capabilities provided by ORMs.
Learning Curve: Teams may find it harder to get started with domain-driven design principles.
Guidance for Teams

To guide teams in maintaining a clean separation between persistence concerns and the core business logic, consider these strategies:

Define Clear Boundaries: Establish clear guidelines on what constitutes domain logic versus persistence logic.
Use Repositories and Mappers: Encourage the use of repositories to abstract database access while using mappers to bridge the gap between domain objects and ORM entities if needed.
Implement Separation of Concerns: Ensure that domain models are purely focused on business rules, leaving storage details to specialized classes.
Refactor Regularly: Regular code reviews and refactoring sessions can help keep the boundaries clear.

By adopting a strategic approach, you can leverage the strengths of both ORM entities and pure domain objects while mitigating their respective drawbacks. This balanced design ensures that your application remains maintainable, scalable, and robust over time.

#### Common Mistakes
to Avoid
Ignoring Schema Changes: Frequent database schema changes without corresponding code updates can lead to bugs and inconsistencies.
Overuse of ORM Features: Over-reliance on ORM features for business logic can blur the lines between domain and persistence concerns.
Neglecting Testing: Poorly defined testing strategies can make it harder to detect issues related to data access.

**Comparison:**

```
Approach	Benefits	Costs
ORM	Simple, automatic mapping	Tight coupling, maintenance overhead
Domain	Loose coupling, maintainable	Complexity, performance considerations
```

**Code:**

```python
# Example of a simple repository pattern in Python

class UserRepository:
    def __init__(self, session):
        self.session = session

    def get_user_by_id(self, user_id):
        return self.session.query(User).filter_by(id=user_id).first()

    def save_user(self, user):
        self.session.add(user)
        self.session.commit()

# Example of a pure domain object in Python

class User:
    def __init__(self, id, name, email):
        self.id = id
        self.name = name
        self.email = email

    def send_email(self, message):
        # Business logic for sending an email
        pass
```

**Example input / output:**

```
Input: User with id=3 and name="Charlie"
Output: User object with updated fields
PYTHON
Copy
# Given input
user_input = {"id": 3, "name": "Charlie"}

# Processing
repo = UserRepository(session)
new_user = User(user_input['id'], user_input['name'], "")

# Business logic
new_user.send_email("Welcome message")

# Save to database
repo.save_user(new_user)
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@example.com"}
  ]
}
```

**Flow:**

1. Domain Model
2. Persistence Layer
3. Database Schema
4. Testability
5. Reasoning about businesslogic
6. Maintain schema changesseparately
7. Brittleness if not managedcarefully

