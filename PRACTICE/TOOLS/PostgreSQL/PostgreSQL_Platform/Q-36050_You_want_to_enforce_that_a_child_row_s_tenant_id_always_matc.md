---
type: "practice"
difficulty: "Deep"
---

# You want to enforce that a child row's tenant_id always matches its parent's tenant_id across a foreign key; how would you model a composite foreign key to guarantee tenant integrity and why is this better than trusting the app?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Adding Constraints Online](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Adding_Constraints_Online.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a child row's tenant_id always matches its parent's tenant_id in PostgreSQL, we need to model a composite foreign key. This approach guarantees tenant integrity by enforcing referential constraints directly at the database level, rather than relying on application logic. Here’s how we can achieve this:

#### Overview
We will define a composite foreign key constraint that ensures every child row references a valid parent row with the same tenant_id. By doing so, we offload the responsibility from the application to the database, making our system more robust and easier to maintain.

#### Triage
The question is about enforcing referential integrity between related tables using composite foreign keys in PostgreSQL. Specifically, it asks us to model a foreign key where both parent_id and tenant_id are required to match corresponding columns in the parent table. We need to consider the implications of this approach, such as performance overhead and potential complexity.

#### First-Principles Solution

Define Tables: Let’s assume we have two tables: parents (with columns id, tenant_id) and children (with columns id, parent_id, tenant_id).
Create Composite Foreign Key Constraint:
SQL
ALTER TABLE children ADD CONSTRAINT fk_parent_tenant_id
    FOREIGN KEY (parent_id, tenant_id)
    REFERENCES parents(id, tenant_id);
Explanation: This constraint ensures that any insertion or update to a children row with an invalid tenant_id will be blocked. It aligns the database schema closely with our business rules.

#### Closing Summary
and Common Mistakes/Follow-Ups:

By using a composite foreign key, we ensure that data integrity is enforced at the database level, which can prevent bugs caused by application logic failures.
However, this approach might introduce some overhead in terms of query performance due to additional indexing requirements. It’s important to balance these concerns with the overall system architecture.

