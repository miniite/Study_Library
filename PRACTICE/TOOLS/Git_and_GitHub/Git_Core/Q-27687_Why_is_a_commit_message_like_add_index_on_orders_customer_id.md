---
type: "practice"
difficulty: "Basic"
---

# Why is a commit message like "add index on orders.customer_id to speed up lookups" more useful than "fix db" for a schema change?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When making changes to a database schema, it's crucial to provide clear and detailed commit messages to help other developers understand what was changed and why. A commit message like "add index on orders.customer_id to speed up lookups" is more useful than "fix db" for several reasons.

#### Triaging the Question

The core of this question revolves around understanding the importance of descriptive commit messages in version control systems, specifically using Git and GitHub. The interviewer wants to ensure that you understand how clear and specific information in your commits can help in maintaining a clean codebase and facilitating collaboration among team members.

Key Idea: Descriptive Commit Messages

Commit messages are like documentation for your changes. A well-written commit message provides context about the change, why it was made, and what impact it has on the system. This is particularly important when dealing with schema changes in databases, as these can significantly affect application performance and data integrity.

Reasoning From First Principles

Clarity: "add index on orders.customer_id to speed up lookups" clearly states the action taken (adding an index) and the reason for it (to improve lookup times). This makes it easier for others to understand the change at a glance.

Reproducibility: Detailed commit messages make it easy for anyone who needs to revert or reproduce the changes later. For example, if you need to reapply this index in another environment, having the exact command or SQL statement in the commit message is invaluable.

Traceability: Clear commit messages help trace back the history of a particular change. If an issue arises due to adding this index, it will be easy to identify and roll back the specific changes without affecting other parts of the database schema.

Maintainability: Over time, team members might forget about certain details or context behind a change. Descriptive commit messages ensure that even months after making a change, everyone can understand what was done and why.

#### Example

Let's consider an example where we need to add an index on the customer_id column in the orders table:

SQL
-- add_index_on_customer_id.sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);

The corresponding Git commit message would be:

CODE
add index on orders.customer_id to speed up lookups

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, descriptive commit messages like "add index on orders.customer_id to speed up lookups" are more useful than generic ones like "fix db" because they provide clear context, ease of reproduction, traceability, and maintainability. Common mistakes include using vague or no commit messages at all, which can lead to confusion and inefficiency in development processes.

#### Follow-up
questions might explore scenarios where you need to manage multiple schema changes simultaneously or how to handle complex database operations that involve multiple steps.

**Code:**

```sql
-- add_index_on_customer_id.sql
CREATE INDEX idx_orders_customer_id ON orders (customer_id);
```

