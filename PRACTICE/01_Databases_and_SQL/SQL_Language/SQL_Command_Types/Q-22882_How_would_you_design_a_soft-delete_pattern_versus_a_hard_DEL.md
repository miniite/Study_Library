---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a soft-delete pattern versus a hard DELETE, and what are the query, index, and compliance trade-offs?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ DELETE vs TRUNCATE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DELETE_vs_TRUNCATE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database schema, the choice between a soft delete and a hard delete can have significant implications on how queries perform, indexes are utilized, and regulatory requirements are met. In this answer, I'll first summarize the key differences, then delve into the trade-offs related to query performance, indexing strategies, and compliance considerations.

TL;DR
Soft Delete: Uses an additional column (e.g., is_active) to mark rows as deleted instead of physically removing them.
Hard Delete: Physically removes records from the database, which is simpler but can lead to data loss if not handled carefully.

#### Trade-offs
Query Performance: Soft delete requires extra logic in queries, while hard delete might simplify query construction.
Indexing: Soft delete may impact index size and maintenance, whereas hard delete doesn’t affect indexes directly.
Compliance: Soft delete offers more control over data retention, which can be crucial for compliance.

#### Triaging the Question

The question is probing your understanding of different deletion techniques in database management. Specifically, it aims to assess how you would balance operational efficiency (query performance) with data integrity and legal requirements (compliance). We will assume that both methods are implemented using standard SQL statements and consider a typical use case involving customer records.

#### First-Principles Solution
1. Soft Delete Pattern

In the soft delete pattern, instead of removing rows from the table, you set an is_active flag or similar column to indicate whether a row is active (1) or inactive (0). This approach allows for easy restoration or archival of data.

SQL
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

To delete a customer, you would set the is_active column to false:

SQL
UPDATE customers SET is_active = FALSE WHERE id = 1;
2. Hard Delete Pattern

In contrast, a hard delete physically removes rows from the table using the DELETE statement:

SQL
DELETE FROM customers WHERE id = 1;
Query Performance Trade-offs
Soft Delete: Requires additional logic in queries to filter out inactive records.
Hard Delete: Simpler query logic but can lead to data loss if not handled carefully.

For example, when retrieving active customers:

Soft Delete:
SQL
SELECT * FROM customers WHERE is_active = TRUE;
Hard Delete:
SQL
SELECT * FROM customers;

The soft delete approach requires filtering out inactive rows in every query that retrieves data. This can be more complex and potentially slower, depending on the number of active records.

Indexing Trade-offs
Soft Delete: Setting is_active to false may affect index size if the column is part of any indexes.
Hard Delete: Does not impact indexing as rows are physically removed.

For instance, if the id and is_active columns are included in an index:

SQL
CREATE INDEX idx_customers ON customers (id, is_active);

In a soft delete scenario, you may need to update this index after setting is_active = FALSE.

Compliance Trade-offs
Soft Delete: Offers more control over data retention and can be easier to comply with regulatory requirements.
Hard Delete: Simplifies data management but may result in permanent data loss if not backed up properly.

Compliance often requires documentation of data deletion policies. Soft delete allows you to log deletions, ensuring transparency and traceability. For example:

SQL
INSERT INTO deleted_customers (id, name, email, deleted_at) VALUES (1, 'John Doe', 'johndoe@example.com', CURRENT_TIMESTAMP);

#### Closing Summary
+ Common Mistakes/Follow-ups

In summary, soft delete provides more control over data retention and can be easier to comply with regulatory requirements. However, it requires additional logic in queries and may impact indexing strategies. Hard delete is simpler but risks permanent data loss if not handled carefully.

#### Common mistakes
include:

Overlooking the need for additional query filtering.
Failing to manage indexes correctly after soft deletes.
Ignoring compliance and audit log requirements when using hard delete.

#### Follow-up
questions might explore specific use cases, such as restoring soft-deleted records or handling large-scale deletions in both patterns.

**Comparison:**

| Customer ID | Name | Email | Created At | Updated At | Active | |-------------|-----------|-------------------|-----------------|----------------| | 1 | John Doe | johndoe@example.com | 2023-10-01 14:30 | 2023-10-01 15:00 | TRUE | | 2 | Jane Smith| smithjane@example.com| 2023-10-02 16:00 | 2023-10-02 17:00 | FALSE |

**Code:**

```sql
-- Soft Delete Example
CREATE TABLE customers (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN NOT NULL DEFAULT TRUE
);

UPDATE customers SET is_active = FALSE WHERE id = 1;

-- Hard Delete Example
DELETE FROM customers WHERE id = 1;
```

**Example input / output:**

```text
-- Soft Delete Query Example
SELECT * FROM customers WHERE is_active = TRUE;

-- Hard Delete Query Example
SELECT * FROM customers;
```

**Data:**

```sql
CREATE TABLE deleted_customers (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    deleted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
INSERT INTO deleted_customers (id, name, email) VALUES (1, 'John Doe', 'johndoe@example.com');
```

**Flow:**

1. Yes
2. No
3. User Requests Deletion
4. Soft Delete?
5. Set is_active = False
6. Physically Remove Row
7. Log to Deleted CustomersTable
8. Commit Transaction

