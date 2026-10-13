---
type: "practice"
difficulty: "Intermediate"
---

# How do you reason about the complexity of a bulk-upsert endpoint that must be efficient and idempotent, comparing per-row queries (O(n) round-trips) to a single batched statement?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Sorting Cost in Services](<../../../../08_Algorithms_and_Data_Structures/Core/Complexity_Analysis/Sorting_Cost_in_Services.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the efficiency and idempotence requirements of a bulk-upsert endpoint, we need to balance time complexity with space complexity. The goal is to efficiently handle multiple records while ensuring that duplicates are managed correctly without causing any side effects.

#### Triage

The question asks us to compare two approaches:

Per-Row Queries (O(n) round-trips): Each insert or update operation is handled individually, which means n separate database operations.
Single Batched Statement: A single SQL statement that processes all records at once, reducing the number of round-trips.

Both methods have different trade-offs in terms of time and space complexity. We need to consider how idempotence can be achieved in each scenario and evaluate their overall efficiency.

#### Assumptions
Idempotence Requirement: The operation must ensure that inserting or updating a record multiple times does not change the outcome after the first successful execution.
Database Constraints: We will assume a relational database with primary key constraints to enforce uniqueness.
Error Handling: We need to handle cases where some records might fail due to existing duplicates.

#### First Principles Solution
Per-Row Queries (O(n) round-trips)

Check for Existence and Insert/Update:

For each record, check if it exists in the database.
If it does not exist, insert it.
If it already exists, update it.

Idempotence:

Each operation is atomic, so re-executing a failed transaction will not change the outcome after the first successful execution.

Time Complexity: O(n) round-trips, where n is the number of records to upsert.

Space Complexity: O(1) per record (excluding database space for storing records).

Single Batched Statement

Single Insert/Update Statement:

#### Cons
truct a single SQL statement that includes all records and uses a conditional operation like ON CONFLICT in PostgreSQL or UPSERT in MySQL.

Idempotence:

The database handles conflicts internally, ensuring no duplicates are inserted if they already exist.

Time Complexity: O(1) round-trip (one batched statement), but the processing time for the database could be higher due to complex queries.

Space Complexity: O(n) for storing all records in a single statement.

Comparing Both Approaches
Time Complexity
Per-row approach: O(n)
Batched approach: O(1)

The batched approach is more efficient in terms of round-trips, which can be significant for large datasets. However, the processing time per batch might be higher due to the complexity of handling all records at once.

Space Complexity
Per-row approach: O(1) per record (excluding storage)
Batched approach: O(n)

The batched approach uses more memory for storing all records in a single statement, which could be a concern if dealing with very large datasets.

Idempotence

Both approaches are idempotent by design. The per-row approach ensures atomicity through individual transactions, while the batched approach relies on database constraints and conditional operations to handle conflicts.

#### Closing Summary

Given the requirements for efficiency and idempotence, the single batched statement is generally more efficient in terms of round-trips but may have higher processing time. It’s crucial to consider the specific use case and dataset size when deciding between these approaches.

#### Common Mistakes/Follow-Ups
Ignoring Database Constraints: Not utilizing database constraints like ON CONFLICT can lead to duplicate entries.
Overlooking Error Handling: Failing to handle partial success cases where some records are upserted but others fail.
Not Considering Batch Size: In a batched approach, the size of the batch could impact performance and should be tuned for optimal results.

**Comparison:**

| Approach   | Time Complexity | Space Complexity | Idempotence |
|------------|-----------------|------------------|-------------|
| Per-row    | O(n)            | O(1)             | Yes         |
| Batched    | O(1)            | O(n)             | Yes         |

**Code:**

```sql
-- Example of a single batched statement using ON CONFLICT in PostgreSQL
INSERT INTO users (id, name, email)
VALUES ('1', 'Alice', 'alice@example.com'), ('2', 'Bob', 'bob@example.com')
ON CONFLICT (id)
DO UPDATE SET name = EXCLUDED.name, email = EXCLUDED.email;
```

**Example input / output:**

```

Given the input:

JSON
Copy
[
  {"id": "1", "name": "Alice", "email": "alice@example.com"},
  {"id": "2", "name": "Bob", "email": "bob@example.com"}
]

The batched statement would execute a single SQL query, potentially reducing the round-trips.
```

**Data:**

```json
[
  {"id": "1", "name": "Alice", "email": "alice@example.com"},
  {"id": "2", "name": "Bob", "email": "bob@example.com"}
]
```

**Flow:**

1. Batched
2. Per-Row
3. Insert
4. Update
5. UPSERT
6. Per-Row Queries
7. Check for Existence
8. Insert Record
9. Update Record
10. Single Batched Statement
11. Handle Conflicts
12. Batch Upsert Records
13. Atomic Operation
14. Efficient Processing

