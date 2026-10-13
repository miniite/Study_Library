---
type: "practice"
difficulty: "Deep"
---

# Explain a DEFERRABLE INITIALLY DEFERRED constraint and give a real case, like swapping two rows' unique sort positions, where deferring the check until COMMIT is the only clean solution.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Unique Constraints for Dedup](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Unique_Constraints_for_Dedup.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A DEFERRABLE INITIALLY DEFERRED constraint in SQL allows us to delay the validation of a constraint condition until the transaction commits (i.e., at the end, when you explicitly call COMMIT). This is particularly useful in scenarios where immediate validation isn't feasible or desirable, but post-commit checks are required for data consistency.

#### Triaging the Question

The interviewer is probing your understanding of transactional constraints and their behavior within SQL databases. Specifically, they want to know how you can implement a constraint that defers its enforcement until the end of a transaction, and provide an example where this approach is necessary.

Core Concept and Assumptions
We are dealing with a database management system (DBMS) that supports DEFERRABLE INITIALLY DEFERRED constraints.
The context here involves swapping two rows' unique sort positions, which must maintain data integrity but cannot be checked immediately due to the nature of the operation.
Detailed Solution

Let's consider a scenario where we have an application managing a sorted list of items. Each item has a sort_order field that determines its position in this list. We want to swap the positions of two items, but ensuring that no other constraints are violated during the intermediate state is crucial.

Step 1: Setup and Initial Constraints

First, let's define our table with an id, name, and sort_order column:

SQL
CREATE TABLE items (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    sort_order INT UNIQUE NOT NULL
);

In this case, the sort_order field is unique and should not have duplicate values. However, during the swap operation, we might temporarily violate this constraint.

Step 2: Defining the DEFERRABLE Constraint

To handle this situation, we can add a deferrable constraint:

SQL
ALTER TABLE items ADD CONSTRAINT unique_sort_order CHECK (sort_order IS NOT NULL);

We set this constraint DEFERRABLE INITIALLY DEFERRED to allow us to temporarily bypass it during transactions.

Step 3: Implementing the Swap Operation

Now, let's implement a function that swaps two rows' sort orders:

SQL
BEGIN TRANSACTION;

-- Temporarily increase or decrease sort_order values to violate the constraint
UPDATE items SET sort_order = 10 WHERE id = 1; -- Assume original order is 2 and we need to temporarily make it 10
UPDATE items SET sort_order = 2 WHERE id = 2;  -- Original order was 3, now becomes 2

-- Perform the actual swap operation that respects transactional integrity
UPDATE items SET sort_order = old_sort_order WHERE id IN (1, 2);
UPDATE items SET old_sort_order = sort_order WHERE id = 1;
UPDATE items SET sort_order = old_sort_order WHERE id = 2;

COMMIT; -- The constraint check is deferred until this point.

In the above script:

We first update both rows' sort_order temporarily, breaking the unique constraint in preparation for the swap.
After performing the swap operation, we commit the transaction. At this point, the DBMS checks the unique_sort_order constraint and enforces it.

#### Closing Summary

Using a DEFERRABLE INITIALLY DEFERRED constraint allows us to temporarily bypass data integrity rules during an operation that requires intermediate changes. In our example, swapping rows' unique sort positions is cleaner when we can defer validation until after the transaction commits. This approach ensures that while the swap is in progress, no immediate violations occur, and only at the end of the transaction do we ensure that the final state adheres to all constraints.

#### Common Mistakes/Follow-Ups
Ignoring Immediate Consistency: Ensuring that you don't violate any other constraints or business rules during the intermediate state.
Overlooking Constraint Enforcement: Failing to properly handle the constraint check at commit time, leading to inconsistent data if not addressed.
Not Considering Transaction Rollbacks: Understanding how ROLLBACK behaves with deferrable constraints and ensuring your application handles these cases appropriately.

**Comparison:**

| id | name | sort_order |
|----|------|------------|
| 1  | A    | 3          |
| 2  | B    | 2          |

**Code:**

```

ALTER TABLE items ADD CONSTRAINT unique_sort_order CHECK (sort_order IS NOT NULL);

CODE
Copy

```sql
BEGIN TRANSACTION;

UPDATE items SET sort_order = 10 WHERE id = 1; -- Temporary update to violate constraint
UPDATE items SET sort_order = 2 WHERE id = 2;

UPDATE items SET sort_order = old_sort_order WHERE id IN (1, 2);
UPDATE items SET old_sort_order = sort_order WHERE id = 1;
UPDATE items SET sort_order = old_sort_order WHERE id = 2;

COMMIT;
```

**Example input / output:**

```text
- **Input:** Initial state with `id=1, name='A', sort_order=2` and `id=2, name='B', sort_order=3`.
- **Output (After Swap):** Final state with `id=1, name='A', sort_order=3` and `id=2, name='B', sort_order=2`.

The constraint is deferred until the commit phase.
```

**Data:**

```sql
CREATE TABLE items (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    sort_order INT UNIQUE NOT NULL
);
```

**Flow:**

1. Start Transaction
2. Update sort_order = 10 forid=1
3. Update sort_order = 2 for id=2
4. Exchange sort_orders for ids(1,2)
5. Commit Transaction
6. Check constraints at committime

