---
type: "practice"
difficulty: "Deep"
---

# Explain why fetching into a %ROWTYPE record from a cursor with a joined query can be brittle when the base table schema evolves.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Ref Cursor Consistency and Limits](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Ref_Cursor_Consistency_and_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with cursors in PL/SQL, especially those that fetch rows from a joined query, there's a significant risk of brittleness if the underlying base table schema changes. This problem stems from how cursor-based operations handle schema evolution without proper consideration for maintaining data consistency and integrity.

#### Triage

The question is probing your understanding of:

How cursors work in PL/SQL.
The impact of schema changes on cursor-based operations.
Best practices to ensure robustness when dealing with evolving schemas.

#### Assumptions
made are:

You're familiar with Oracle PL/SQL and its %ROWTYPE feature for defining records based on database table structures.
Schema evolution involves adding, removing, or altering columns in the base tables.

#### First-Principles Solution

Let's break down a common scenario where this brittleness arises. Consider you have a cursor defined to fetch rows from a joined query:

SQL
CURSOR c1 IS
    SELECT t1.column1, t2.column2 FROM table1 t1 JOIN table2 t2 ON t1.id = t2.id;

Now suppose the schema of table1 changes by adding a new column new_column. The cursor definition remains unchanged. When you attempt to fetch rows into a %ROWTYPE record:

PLSQL
v_record c1%ROWTYPE;

FOR rec IN c1 LOOP
    -- Process rec.column1 and rec.column2 here
END LOOP;

Even though the query in c1 returns two columns, trying to access rec.new_column will result in an error because it doesn't exist. This is a classic example of brittleness.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The core issue lies in how cursors and %ROWTYPE interact with evolving schemas. To avoid such issues, you can adopt several strategies:

Use Explicit Column Selection: Define the cursor to return only specific columns, reducing dependency on schema changes.
Dynamic SQL: Use dynamic SQL constructs like EXECUTE IMMEDIATE to build queries that adapt to schema changes.
Refined Data Validation: Implement validation logic within your procedures to ensure data consistency even when schemas evolve.

#### Common Mistakes
Over-reliance on cursors with %ROWTYPE without considering future schema changes.
Ignoring the dynamic nature of database structures and treating them as static entities.
Not updating code in a timely manner to reflect schema modifications.

If you're asked further, consider discussing:

The trade-offs between static vs. dynamic SQL approaches.
How these strategies impact performance and maintainability.

#### Example
s from other database systems where similar issues arise or are handled differently.

**Comparison:**

| Table Schema | Before Change | After Adding Column |
|--------------|---------------|---------------------|
| table1       | id (PK), column1 | id (PK), column1, new_column |
| table2       | id (PK), column2, column3 | id (PK), column2, column3 |

**Code:**

```
PLSQL
Copy
CURSOR c1 IS
    SELECT t1.column1, t2.column2 FROM table1 t1 JOIN table2 t2 ON t1.id = t2.id;

v_record c1%ROWTYPE;

FOR rec IN c1 LOOP
    -- Process rec.column1 and rec.column2 here
END LOOP;
```

**Example input / output:**

```text
-- Example input and output for dynamic SQL approach
EXECUTE IMMEDIATE 'SELECT t1.column1, t2.column2 FROM table1 t1 JOIN table2 t2 ON t1.id = t2.id' INTO v_record;
```

**Data:**

```sql
-- Example schema before change
CREATE TABLE table1 (
    id NUMBER PRIMARY KEY,
    column1 VARCHAR2(50)
);

CREATE TABLE table2 (
    id NUMBER PRIMARY KEY,
    column2 VARCHAR2(50),
    column3 VARCHAR2(50)
);
```

**Flow:**

1. No
2. Yes
3. Define Cursor with %ROWTYPE
4. Schema Changes?
5. Process Data as Usual
6. Refactor Code to Handle NewColumns
7. No Errors, Stable Application
8. Ensure Consistency andIntegrity Checks

