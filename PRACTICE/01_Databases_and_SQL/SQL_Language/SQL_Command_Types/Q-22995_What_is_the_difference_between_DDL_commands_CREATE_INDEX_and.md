---
type: "practice"
difficulty: "Basic"
---

# What is the difference between DDL commands CREATE INDEX and DROP INDEX?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML Statements](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database management, understanding Data Definition Language (DDL) commands is fundamental. The primary distinction lies in their roles within a database schema. Today, we'll explore CREATE INDEX and DROP INDEX, two critical DDL commands that manage indexes on tables.

#### Triaging the Question

The question asks us to differentiate between CREATE INDEX and DROP INDEX. These commands are part of DDL (Data Definition Language) used for defining or modifying the structure of a database. Specifically, we're looking at operations related to indexing, which is essential for improving query performance.

#### First-Principles Solution

CREATE INDEX: This command creates an index on one or more columns in a table. An index speeds up data retrieval by providing a faster way to locate rows with specific values. When you issue CREATE INDEX, the database generates additional storage that maps keys to their corresponding locations in the table.

DROP INDEX: Conversely, this command removes an existing index from a table. Dropping an index can be useful when it no longer serves its purpose or when there are performance issues associated with maintaining the index.

Summary and Common Mistakes

In summary, CREATE INDEX is used to enhance query performance by creating indexes, while DROP INDEX is used to remove these indexes, which can help in reducing storage space or improving write operations. A common mistake is not considering the impact on query performance when dropping an index.

Code Blocks
SQL
-- Example of CREATE INDEX
CREATE INDEX idx_customer_name ON customers (name);

-- Example of DROP INDEX
DROP INDEX idx_customer_name ON customers;

**Comparison:**

```
Command	Effect
CREATE INDEX	Creates an index on the specified columns.
DROP INDEX	Removes the specified index from the table.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is command CREATE INDEX?
7. Create Index
8. Is command DROP INDEX?
9. Drop Index
10. Command Not Recognized

