---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide between a dependency-injected repository pattern and calling the ORM directly in FastAPI path operations, weighing testability against added indirection?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Database Access and Tool Calls](<../../../../TOOLS/Python/FastAPI/Database_Access_and_Tool_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using a dependency-injected repository pattern versus calling the ORM directly in FastAPI path operations, we need to weigh the benefits of testability and reduced coupling against the potential overhead of added indirection. The key aspects are understanding how each approach impacts code structure, testing, and maintainability.

Firstly, let's triage the question:

We're evaluating two approaches for data access: dependency injection (DI) through repositories versus direct ORM calls.
The primary trade-offs are testability, maintainability, and complexity.
Our assumptions include a FastAPI application with a database-backed model and the ability to inject dependencies.

Now let's delve into the solution from first principles:

Dependency Injection via Repositories
Testability: Using repositories encapsulates data access logic within them, making it easier to mock or replace this layer in unit tests. This separation ensures that your business logic remains clean and decoupled from database specifics.
Maintainability: By keeping data access isolated, changes to the database schema can be handled without altering path operations, promoting better maintainability.
Complexity: While more complex initially due to added abstraction, it enhances modularity and reusability.
Direct ORM Calls
Testability: Direct ORM calls are harder to test as they rely on actual database interactions, complicating unit testing scenarios.
Maintainability: Changes in the database schema may require updates throughout path operations if direct ORM calls are used extensively.
Complexity: Simpler setup initially but can become cumbersome and repetitive when dealing with complex queries.
Step-by-Step Solution
Assess Project Complexity: For simple, small-scale projects where performance is not a critical concern, direct ORM calls might suffice due to simplicity.
Evaluate Testing Requirements: If you prioritize robust testing, particularly for unit tests and integration tests, repository patterns offer significant benefits.

#### Cons
ider Future Scalability: In larger or more complex applications, repositories provide better scalability and maintainability over time.

#### Closing Summary
+ Common Mistakes

In summary, the choice between dependency-injected repositories and direct ORM calls in FastAPI depends on your project's specific needs. For most intermediate to large-scale projects where testability and future maintenance are priorities, repositories offer substantial advantages. However, for smaller or very performance-critical applications, the simplicity of direct ORM calls might be preferable.

#### Common mistakes
include:

Over-engineering small projects by introducing unnecessary abstractions.
Ignoring the importance of testing in favor of simpler but less maintainable code.
Underestimating future requirements and not planning for potential scalability issues.

**Comparison:**

```
Approach	Testability	Maintainability	Complexity
Repositories	High	High	Medium-High
Direct ORM Calls	Low	Low-Medium	Low
```

**Code:**

```python
# Example of using a repository pattern with FastAPI
from fastapi import Depends, FastAPI
from sqlalchemy.orm import Session
from repositories.user_repository import UserRepository

app = FastAPI()

def get_db_session():
    # Dependency to provide database session
    pass

@app.get("/users/{user_id}")
async def read_user(user_id: int, db: Session = Depends(get_db_session)):
    user_repo = UserRepository(db)
    return user_repo.get_user_by_id(user_id)

# Example of direct ORM call
from fastapi import FastAPI
from sqlalchemy.orm import Session
from models.user_model import User

app = FastAPI()

@app.get("/users/{user_id}")
async def read_user(user_id: int, db: Session):
    return db.query(User).filter_by(id=user_id).first()
```

**Example input / output:**

```text
Input:
GET /users/1

Output:
{
  "id": 1,
  "name": "John Doe",
  "email": "john.doe@example.com"
}
```

**Data:**

```json
{
  "user": {
    "id": 1,
    "name": "John Doe",
    "email": "john.doe@example.com"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Repository Pattern
6. Is Testability Critical?
7. Test Repository Layer
8. Simplicity Sufficient
9. Direct ORM Calls
10. Simplicity Required?
11. Direct ORM Calls
12. Test Directly

