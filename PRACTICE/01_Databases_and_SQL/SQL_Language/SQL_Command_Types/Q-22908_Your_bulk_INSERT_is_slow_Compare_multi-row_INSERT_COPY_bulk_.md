---
type: "practice"
difficulty: "Intermediate"
---

# Your bulk INSERT is slow. Compare multi-row INSERT, COPY/bulk load, and prepared statements, and explain when each wins.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with performance issues in a database, understanding how to efficiently insert large amounts of data can be crucial. In this scenario, you need to compare three common methods: multi-row INSERT, COPY/bulk load, and prepared statements. Each method has its own strengths and weaknesses, which we will explore from first principles.

Multi-row INSERT is straightforward but can suffer from overhead due to transaction management and individual row validation. COPY or bulk load is designed for high-speed data insertion, often bypassing many of the constraints and validations that slow down multi-row INSERT. Prepared statements are reusable templates that can help reduce compilation time in complex SQL operations.

Let's dive into each method step by step:

Multi-Row INSERT: This involves sending multiple rows in a single SQL statement to the database.
COPY/ Bulk Load: This method uses a file or standard input as a source of data and directly loads it into the table, often bypassing the transactional overhead.
Prepared Statements: These are pre-compiled SQL statements that can be executed multiple times with different parameters.

**Comparison:**

| Method            | Description                                  | Pros                         | Cons                      |
|-------------------|----------------------------------------------|------------------------------|----------------------------|
| Multi-row INSERT  | Insert multiple rows in a single statement    | Simple to implement           | High overhead due to ACID  |
| COPY/bulk load    | Load data from a file or standard input       | Very fast                    | File format and permissions required |
| Prepared Statement| Pre-compile SQL statements for reuse         | Reduce compilation time      | Additional complexity in setup |

**Code:**

```sql
-- Multi-row INSERT example
INSERT INTO users (id, name, email) VALUES
(1, 'Alice', 'alice@example.com'),
(2, 'Bob', 'bob@example.com'),
(3, 'Charlie', 'charlie@example.com');

-- COPY/bulk load example using a file
\copy users FROM '/path/to/data.csv' DELIMITER ',' CSV HEADER;

-- Prepared statement example in PostgreSQL
PREPARE bulk_insert AS INSERT INTO users (id, name, email) VALUES ($1, $2, $3);
EXECUTE bulk_insert(4, 'David', 'david@example.com');
EXECUTE bulk_insert(5, 'Eve', 'eve@example.com');
```

**Example input / output:**

```
Multi-row INSERT:
SQL
Copy
-- Input
INSERT INTO users (id, name, email) VALUES
(1, 'Alice', 'alice@example.com'),
(2, 'Bob', 'bob@example.com'),
(3, 'Charlie', 'charlie@example.com');

-- Output: Records inserted successfully.
COPY/bulk load:
SQL
Copy
\copy users FROM '/path/to/data.csv' DELIMITER ',' CSV HEADER;

-- Assuming the file contains:
# id,name,email
1,Alice,alice@example.com
2,Bob,bob@example.com
3,Charlie,charlie@example.com

-- Output: Records inserted successfully.
Prepared Statement:
SQL
Copy
PREPARE bulk_insert AS INSERT INTO users (id, name, email) VALUES ($1, $2, $3);

EXECUTE bulk_insert(4, 'David', 'david@example.com');
EXECUTE bulk_insert(5, 'Eve', 'eve@example.com');

-- Output: Records inserted successfully.
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@example.com"},
    {"id": 3, "name": "Charlie", "email": "charlie@example.com"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Can be slow due to ACIDconstraints
4. Yes
5. No
6. Start
7. Is data large?
8. COPY/bulk load
9. Multi-row INSERT
10. Prepare for high concurrency
11. Use prepared statements forefficiency
12. Ensure file permissions andformat are correct
13. Data loaded into the database
14. Is there a need for dynamicparameterization?

