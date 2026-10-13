---
type: "practice"
difficulty: "Basic"
---

# What is a unique index and how does it help enforce that your dimension table has no duplicate business keys after a load?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Adding Unique Indexes Safely](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Adding_Unique_Indexes_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A unique index ensures that each value in a specific column (or columns) of a database table is distinct and guarantees there are no duplicates. In the context of loading data into a dimension table, a unique index on the business key enforces uniqueness after the load process. This prevents any duplicate records from being inserted, maintaining data integrity.

#### Triaging the Question

The question asks us to explain what a unique index is and how it helps prevent duplicates in a dimension table during data loads. The interviewer is likely probing your understanding of database indexing and its role in ensuring data quality and integrity.

#### First Principles Solution

Understanding Unique Indexes:

A unique index on a column or columns in a database ensures that each value in those columns is distinct. No two rows can have the same combination of values for the indexed columns.
It's typically created using an SQL UNIQUE constraint, which enforces data integrity rules.

Business Keys and Dimension Tables:

A dimension table stores categorical attributes (e.g., customer ID, product ID) that are used to categorize or classify other tables in a data warehouse.
The primary key of the dimension table is often referred to as the business key, which uniquely identifies each row.

Enforcing Uniqueness with Unique Indexes:

When you load data into a dimension table and want to ensure that no duplicates are inserted, you can create a unique index on the business key column.
During the insertion process, if the database encounters an attempt to insert a value that already exists in the unique index, it will either throw an error or prevent the insert operation, depending on the configuration.

#### Implementation
Example:

SQL
CREATE UNIQUE INDEX idx_customer_id ON customers (customer_id);
This SQL statement creates a unique index named idx_customer_id on the customer_id column in the customers table.
If you try to insert a customer with an already existing customer_id, the database will enforce this constraint and prevent the operation.

Benefits:

Data Integrity: Ensures that no duplicate records are inserted, maintaining data consistency.
Performance: Efficiently enforces uniqueness without requiring full table scans during insert operations.
Error Prevention: Helps catch issues early in the data loading process before they propagate to other parts of the database.

#### Closing Summary
+ Common Mistakes/Follow-Ups:
A unique index is a crucial tool for maintaining data integrity and preventing duplicate records. It's particularly important when dealing with dimension tables, where each record must be unique.
Common Mistake: Not applying unique constraints or indexes to critical columns can lead to data redundancy and inconsistencies.

#### Follow-Up
Questions:
How would you handle cases where the business key needs to allow null values?
Can a composite index (unique on multiple columns) serve similar purposes?

**Comparison:**

```
tableDiagram
    customers -->|customer_id| idx_customer_id
```

**Code:**

```sql
-- Creating a unique index
CREATE UNIQUE INDEX idx_customer_id ON customers (customer_id);
```

**Example input / output:**

```text
-- Attempting to insert a duplicate customer ID
INSERT INTO customers (customer_id, name) VALUES (1, 'Dave');
-- Error:
-- ERROR:  duplicate key value violates unique constraint "idx_customer_id"
-- DETAIL: Key (customer_id)=(1) already exists.
```

**Data:**

```json
{
  "customer_id": [1, 2, 3, null],
  "name": ["Alice", "Bob", "Charlie", null]
}
```

**Flow:**

1. No
2. Yes
3. Data Load Starts
4. Check for Duplicate CustomerID
5. Duplicate Found?
6. Insert New Record
7. Error: Duplicate Key Violation

