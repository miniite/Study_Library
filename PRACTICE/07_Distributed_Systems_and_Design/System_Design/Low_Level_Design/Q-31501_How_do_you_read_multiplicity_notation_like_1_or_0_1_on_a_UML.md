---
type: "practice"
difficulty: "Basic"
---

# How do you read multiplicity notation like 1..* or 0..1 on a UML association?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Domain Event Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Tenant_Isolation_in_Repositories/Domain_Event_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When reading multiplicity notation in a UML association, it's crucial to understand what each number represents and how they interact with entities. The numbers denote cardinality—how many instances of one entity can be associated with another. Let’s break down the common notations like 1..*, 0..1, etc.

**Code:**

```text
# Example UML association
class User {
    + id: int
    + name: string
}

class Post {
    + id: int
    + title: string
    - author: User (optional)
}
```

**Example input / output:**

```text
For the multiplicity `1..*`:
- This means that one instance of the first entity can be associated with zero or more instances of the second entity. For example, a `User` can have 0 to many `Posts`.

For the multiplicity `0..1`:
- This indicates that an instance of the first entity is optional and can be associated with at most one instance of the second entity. For example, a `Post` has zero or one `author`.
```

