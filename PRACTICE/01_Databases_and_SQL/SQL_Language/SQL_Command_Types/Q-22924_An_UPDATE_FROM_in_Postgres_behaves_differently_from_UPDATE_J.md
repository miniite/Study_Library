---
type: "practice"
difficulty: "Deep"
---

# An UPDATE ... FROM in Postgres behaves differently from UPDATE ... JOIN in MySQL. Explain the semantic difference and a bug it can cause.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the semantic differences between UPDATE ... FROM in PostgreSQL and UPDATE ... JOIN in MySQL, we need to understand how these SQL constructs work under the hood and their intended use cases. Let's break down the core concepts first.

TL;DR

The main difference lies in the way they handle multiple tables during an update operation. UPDATE ... FROM in PostgreSQL can be ambiguous, potentially leading to unexpected results or bugs, while UPDATE ... JOIN in MySQL clearly specifies how to join and update rows from two tables.

#### Triaging the Question
What is Being Probed

The question probes your understanding of SQL standards, specifically DDL (Data Definition Language), DML (Data Manipulation Language), DCL (Data Control Language), and TCL (Transaction Control Language). It also tests your knowledge of how different database systems implement these constructs.

#### Assumptions
We are comparing PostgreSQL's UPDATE ... FROM syntax with MySQL's UPDATE ... JOIN syntax.
The focus is on the semantics and potential pitfalls when using these constructs in practice.

#### First Principles Solution
PostgreSQL: UPDATE ... FROM

PostgreSQL allows you to use a FROM clause in an UPDATE statement, which essentially performs a self-join within the same table. For example:

SQL
UPDATE customers c1
SET email = c2.email
FROM (
    SELECT first_name, last_name, email
    FROM customers
) AS c2
WHERE c1.first_name = c2.first_name AND c1.last_name = c2.last_name;

In this case, PostgreSQL performs a join between customers and an inline table defined by the subquery. The join condition is implicitly created based on the equality of columns.

MySQL: UPDATE ... JOIN

MySQL's UPDATE ... JOIN syntax explicitly specifies how to update rows from two tables. For example:

SQL
UPDATE customers c1
JOIN (
    SELECT first_name, last_name, email
    FROM customers
) AS c2 ON c1.first_name = c2.first_name AND c1.last_name = c2.last_name
SET c1.email = c2.email;

Here, the join is explicitly defined using JOIN, and the update sets the column values based on the matching rows.

Semantic Difference

The key difference lies in how ambiguity is handled. PostgreSQL's UPDATE ... FROM can be ambiguous because it performs a self-join without an explicit ON clause. If there are multiple rows that match, PostgreSQL may return an error or behave unpredictably depending on its internal logic.

On the other hand, MySQL's UPDATE ... JOIN requires an explicit join condition (ON clause), which makes the intent clear and avoids potential ambiguities.

#### Common Mistakes
Misunderstanding the Join Condition: Failing to define a proper join condition can lead to incorrect updates or errors.
Ambiguity in PostgreSQL: Not realizing that UPDATE ... FROM without an explicit ON clause can cause issues, as it may not behave as expected.

#### Closing Summary
and Follow-Ups

In summary, while both PostgreSQL's UPDATE ... FROM and MySQL's UPDATE ... JOIN allow for updating rows based on conditions from another table, the explicitness of the join condition in MySQL helps avoid ambiguity. PostgreSQL’s implicit self-join can lead to unexpected results if not used carefully.

#### Follow-Ups
Handling Ambiguity: How would you handle ambiguous updates in PostgreSQL?

#### Performance Considerations
What are the performance implications of using UPDATE ... JOIN versus UPDATE ... FROM?
Best Practices: When should one prefer UPDATE ... JOIN over UPDATE ... FROM, and vice versa?

**Comparison:**

  | id | first_name | last_name | email |
  |---|-----------|----------|-------|
  | 1 | John      | Doe      | john.doe@old.com  |
  | 2 | Jane      | Smith    | jane.smith@old.com|

**Code:**

```sql
-- PostgreSQL Example
UPDATE customers c1
SET email = c2.email
FROM (
    SELECT first_name, last_name, email
    FROM customers
) AS c2
WHERE c1.first_name = c2.first_name AND c1.last_name = c2.last_name;

-- MySQL Example
UPDATE customers c1
JOIN (
    SELECT first_name, last_name, email
    FROM customers
) AS c2 ON c1.first_name = c2.first_name AND c1.last_name = c2.last_name
SET c1.email = c2.email;
```

**Example input / output:**

```text
-- Input in PostgreSQL
UPDATE customers c1
SET email = c2.email
FROM (
    SELECT first_name, last_name, email
    FROM customers
) AS c2
WHERE c1.first_name = c2.first_name AND c1.last_name = c2.last_name;

-- Output (assuming no changes)
No rows affected.
```

**Data:**

```json
{
  "customers": [
    {"id": 1, "first_name": "John", "last_name": "Doe", "email": "john.doe@old.com"},
    {"id": 2, "first_name": "Jane", "last_name": "Smith", "email": "jane.smith@old.com"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is join condition explicit?
5. Use MySQL's UPDATE ... JOIN
6. PostgreSQL's ambiguousself-join may cause issues
7. Ensure performance isoptimized with correct indexes
8. Troubleshoot and documentpotential bugs

