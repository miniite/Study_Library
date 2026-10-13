---
type: "practice"
difficulty: "Deep"
---

# How would you enforce a conditional uniqueness rule, e.g. only one 'active' subscription per user, using a partial/filtered unique index?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Unique and Deferred Constraints](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Unique_and_Deferred_Constraints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a conditional uniqueness rule such as ensuring that each user has only one active subscription, we can use a partial or filtered unique index in SQL. This approach leverages indexes to maintain data integrity efficiently without compromising performance too much. In this case, we'll create an index on the user_id and is_active columns, ensuring that there is at most one row with is_active = true for any given user.

#### Triaging the Question

The core of the question revolves around understanding how to enforce a business rule using database constraints. Specifically, it asks about leveraging indexes to achieve conditional uniqueness. The key aspects include:

Understanding the business requirement: ensuring only one active subscription per user.
Exploring SQL capabilities: Using partial unique indexes to enforce this constraint.
Discussing trade-offs and limitations.

#### First Principles Solution

First, let's define our table structure. Assume we have a subscriptions table with columns such as subscription_id, user_id, start_date, end_date, and is_active.

SQL
CREATE TABLE subscriptions (
    subscription_id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP,
    is_active BOOLEAN DEFAULT FALSE,
    -- Other columns...
);

To enforce the conditional uniqueness rule, we can create a partial unique index on user_id and is_active. This means that for any given user_id, only one row with is_active = true will be allowed.

Step-by-Step Solution
Define the Table: As shown above, ensure your table includes necessary columns.
Create the Partial Unique Index:
SQL
CREATE UNIQUE INDEX idx_user_is_active ON subscriptions (user_id) WHERE is_active = TRUE;
Understanding the Index:
The WHERE clause ensures that the index only enforces uniqueness on rows where is_active = true.
This means for each user, at most one subscription can be active.
Handling Active Subscriptions:
When inserting a new subscription and setting is_active = true, you must check if an existing row with user_id and is_active = true exists before allowing the insertion.
Update Logic:
If an existing subscription needs to be marked as inactive, update its is_active field first, then allow a new active subscription.

#### Closing Summary

By using a partial unique index, we efficiently enforce our business rule of having only one active subscription per user without significantly impacting performance or complexity. The key steps involve defining the table structure and creating an appropriate unique index with a predicate.

#### Common Mistakes
and Follow-Ups
Mistake: Failing to use the WHERE clause in the partial unique index, which would prevent the enforcement of the conditional uniqueness.

#### Follow-Up
Discussing how this approach scales as user numbers increase. Consideration should be given to potential performance implications when dealing with large datasets.

