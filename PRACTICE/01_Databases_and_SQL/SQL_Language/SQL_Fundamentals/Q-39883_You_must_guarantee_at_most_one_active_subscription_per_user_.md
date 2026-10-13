---
type: "practice"
difficulty: "Intermediate"
---

# You must guarantee at most one active subscription per user where inactive is a NULL end_date. Why does a plain UNIQUE constraint fail and how does a partial unique index solve it?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that each user can have at most one active subscription, we need to handle subscriptions with NULL end dates (inactive) and prevent multiple active subscriptions per user. A plain UNIQUE constraint fails because it cannot distinguish between a NULL end_date and an expired date. Instead, using a partial unique index on the combination of user_id and a condition that filters out expired subscriptions is more appropriate.

#### Triage

The question is asking why a simple UNIQUE constraint doesn't work to ensure at most one active subscription per user when the subscription can be inactive (i.e., with a NULL end date). The goal is to understand how a partial unique index can address this issue effectively.

Problem Breakdown

Unique Constraint Fails: A standard UNIQUE constraint on user_id and end_date won't work because:

NULL values in the end_date column are indistinguishable from expired dates.
Multiple rows with the same user_id and a NULL end_date can exist, leading to violations of the unique constraint.

Partial Unique Index: We need an index that enforces the uniqueness only for active subscriptions (i.e., where end_date IS NULL). This approach ensures that each user has at most one active subscription by filtering out expired ones.

#### Solution

Create a Partial Unique Index:

The partial unique index will ensure that there is at most one row with user_id and an inactive subscription (NULL end_date) in the table.

Index Definition:

SQL
CREATE UNIQUE INDEX idx_unique_active_subscription ON subscriptions (user_id) WHERE end_date IS NULL;
Explanation from First Principles

Standard UNIQUE Constraint:

A standard unique constraint on (user_id, end_date) would fail because both end_date being NULL and a valid date can be considered the same for uniqueness purposes. This means we could have multiple rows with the same user_id and NULL end_date.

Partial Unique Index:

By creating an index that includes only the condition WHERE end_date IS NULL, we ensure that each user can have at most one row where their subscription is active (i.e., has a NULL end_date). Any additional rows with the same user_id but non-NULL end_date will be ignored by the unique constraint.

#### Closing Summary

In summary, using a partial unique index allows us to enforce the rule that each user can have at most one active subscription. The standard UNIQUE constraint fails because it cannot distinguish between NULL and expired dates effectively. A partial unique index ensures the uniqueness condition is met only for active subscriptions.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to include the WHERE clause in the index definition, leading to a non-functional constraint.

#### Follow-Up
Discussing potential performance implications of adding an index and whether it's necessary given the query patterns. In some cases, a partial unique index might not be needed if queries do not frequently access active subscriptions directly.

**Code:**

```sql
CREATE UNIQUE INDEX idx_unique_active_subscription ON subscriptions (user_id) WHERE end_date IS NULL;
```

