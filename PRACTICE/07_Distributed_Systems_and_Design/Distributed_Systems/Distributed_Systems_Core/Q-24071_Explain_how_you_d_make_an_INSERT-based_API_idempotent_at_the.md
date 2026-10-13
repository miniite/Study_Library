---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd make an INSERT-based API idempotent at the SQL layer so a client retry after a timeout doesn't create a duplicate row. Compare a unique idempotency-key column with INSERT ... ON CONFLICT versus a separate dedup table.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Ingestion](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To make an INSERT-based API idempotent at the SQL layer, ensuring that a client retrying after a timeout doesn't create duplicate rows, we need to ensure idempotency through either using a unique idempotency key or leveraging the ON CONFLICT clause. Both approaches have their trade-offs and are suitable for different scenarios.

Firstly, let's clarify what idempotence means in this context: it ensures that an operation can be applied multiple times without changing the outcome beyond the first application. In SQL terms, we need to guarantee that a duplicate row is not inserted after a retry.

#### Triage

The core of the question revolves around ensuring INSERT operations are safe and idempotent by either using a unique identifier or employing database features like ON CONFLICT. We will discuss the advantages and disadvantages of each method while considering factors such as performance, complexity, and maintainability.

Unique Idempotency Key

Using a unique idempotency key column involves generating a unique value for every insert request. This approach relies on ensuring that this key is unique across all possible requests, which might not be straightforward if the data being inserted isn't guaranteed to be unique (e.g., an auto-incrementing ID).

Steps:

Generate Idempotency Key: For each API call, generate a unique identifier. This could be based on request headers, timestamps, or a combination of these.
Check Existence Before Insertion: Before executing the INSERT statement, check if the idempotency key already exists in the table.
ON CONFLICT Clause

The ON CONFLICT clause is part of SQL standards and allows for handling conflicts (e.g., duplicate keys) during an insert operation. PostgreSQL, for example, supports this feature through EXCLUDE constraints or INSERT ... ON CONFLICT DO NOTHING.

Steps:

Create an Index with UNIQUE Constraint: Define a unique constraint on the column that you will use as your idempotency key.
Use ON CONFLICT Clause: Write the INSERT statement to include the ON CONFLICT clause, specifying what action to take when a conflict arises.
Comparing Approaches

Idempotency Key Approach:

#### Pros
Flexibility in choosing how to generate unique keys.

#### Cons
Additional logic required for generating and checking the idempotency key.
Potentially complex implementation, especially if dealing with a high volume of requests.

ON CONFLICT Clause Approach:

#### Pros
Simplifies the API logic; you can focus on just inserting data without additional checks.
Built-in support for handling conflicts in SQL databases.

#### Cons
Requires careful design to ensure that unique constraints are appropriately defined and enforced.

#### First-Principles Solution

Let's assume we're using PostgreSQL as our database. We'll create a table with an ON CONFLICT clause to handle the insertions idempotently.

SQL
-- Create the table with a unique constraint on the idempotency key column
CREATE TABLE orders (
    order_id SERIAL PRIMARY KEY,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    customer_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    -- Define an exclusion constraint to ensure idempotency
    UNIQUE (product_id, customer_id)
);

-- Insert data with ON CONFLICT DO NOTHING
INSERT INTO orders (product_id, quantity, customer_id)
VALUES (1, 5, 123)
ON CONFLICT (product_id, customer_id) DO NOTHING;

In this example:

We define a unique constraint on product_id and customer_id, ensuring that each combination of these columns is unique.
The INSERT ... ON CONFLICT DO NOTHING clause prevents the insertion if there's already an existing row with the same product_id and customer_id.

#### Closing Summary

Both methods ensure idempotency, but they differ in implementation complexity. Using a separate idempotency key requires more logic on the application side, while leveraging ON CONFLICT simplifies the API by handling conflicts directly within SQL.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to define the uniqueness constraint properly can lead to accidental duplicates.

#### Follow-Up
Discussing scenarios where one method might be preferred over the other, e.g., if your application is generating unique keys in a distributed environment or if you need more fine-grained control.

