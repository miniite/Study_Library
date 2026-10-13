---
type: "practice"
difficulty: "Deep"
---

# Spanner enforces a mutation limit per transaction and 256 MB row/commit size limits; how do these constraints shape your data model and bulk-write patterns for a high-throughput ingestion pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Query Execution](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data model and bulk-write patterns for a high-throughput ingestion pipeline using Google's Spanner, it’s crucial to understand the constraints imposed by its transactional limits and row/commit size limits. Specifically, transactions are limited to 256 MB in commit size and 10,000 mutations per transaction. This means you need to carefully plan your data model and write patterns to efficiently handle large volumes of data without violating these constraints.

To start, let's break down the key points:

Transaction Size Limit: A single transaction can only process up to 256 MB in a commit.
Mutation Limit: Each transaction can perform up to 10,000 mutations (writes, updates, deletes).
Row/Commit Size Limit: The maximum size of a row is 256 MB.

These constraints significantly impact the design choices you make. Here’s how they shape your data model and bulk-write patterns:

#### Triaging the Question

The question is asking about how to design an efficient ingestion pipeline that respects these Spanner-specific limits, ensuring high throughput while minimizing transaction overhead.

#### First-Principles Solution

Partition Data: To handle large datasets, partition them into smaller chunks that fit within a single transaction's 256 MB limit. For instance, if you have a million rows to insert, you might split them into batches of 10,000 rows each.

Batch Writes: Use batch writes (INSERT/UPDATE/DELETE) rather than individual mutations. Batch operations are more efficient because they reduce the number of network round trips and increase the mutation limit per transaction from 10,000 to 50,000 or even higher.

Row Size Optimization: Ensure that each row does not exceed the 256 MB size limit. This might require normalizing data where appropriate, using efficient data types, and avoiding large binary objects directly in rows.

Concurrency Control: Design your system to allow concurrent ingestion processes, as long as they do not interfere with transactional consistency requirements. Use Spanner's built-in distributed transactions or secondary indexes for coordinating write operations if necessary.

Error Handling and Retry Strategy: Implement a robust error handling mechanism that can retry failed transactions without causing an exponential increase in the commit size. This might involve logging and retrying individual mutations within the same transaction batch.

#### Closing Summary

To achieve high-throughput ingestion with Spanner, you must carefully partition your data into manageable chunks, leverage batch write operations to maximize mutation limits, optimize row sizes, coordinate concurrent writes using appropriate concurrency control mechanisms, and implement error handling strategies that respect transactional boundaries. This approach ensures efficient use of resources while maintaining consistency.

#### Common Mistakes
and Follow-Ups
Ignoring Transaction Limits: Failing to consider the 256 MB commit size limit can lead to inefficient use of transactions.
Overusing Individual Mutations: Not utilizing batch writes can significantly reduce performance, as each mutation incurs additional overhead.
Exceeding Row Size Constraints: Rows that exceed the 256 MB limit will cause errors during commits, leading to failed operations.

#### Follow-ups
might include asking about specific examples or discussing trade-offs between different design choices (e.g., normalized vs. denormalized data models).

**Comparison:**

| Column Name | Type        |
|-------------|-------------|
| user_id     | INT64       |
| email       | STRING(250) |
| first_name  | STRING(100) |
| last_name   | STRING(100) |

**Code:**

```

graph LR; A[Initiate Transaction] --> B1[BEGIN BATCH WRITE]; B1 --> C1[Write 50,000 Rows]; C1 --> D[Commit Batch]; D --> E1[Handle Success/Failure];

CODE
Copy

```plaintext
Input:
- Data to be inserted into Spanner

Output:
- Successful commit of batch write operations
- Error handling for failed transactions or mutations
```

**Example input / output:**

```

Input:

Data to be inserted into Spanner

Output:

Successful commit of batch write operations
Error handling for failed transactions or mutations
CODE
Copy

```yaml
spanner_schema:
  table_name: users
  columns:
    - name: user_id (INT64)
    - name: email (STRING(250))
    - name: first_name (STRING(100))
    - name: last_name (STRING(100))
```

**Flow:**

1. No
2. Yes
3. Start
4. Initiate Transaction
5. Is Batch Size > 256MB?
6. BEGIN BATCH WRITE
7. Write Data
8. Commit Batch Write
9. Handle Success/Failure
10. End
11. Split Data into Smaller Batches
12. BEGIN BATCH WRITE

