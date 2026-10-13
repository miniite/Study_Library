---
type: "practice"
difficulty: "Intermediate"
---

# How would you delete rows from one table based on the existence of matching rows in another table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schemas and Databases](<../../../../TOOLS/MySQL/MySQL_Platform/Schemas_and_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To delete rows from one MySQL table (let's call it TableA) based on the existence of matching rows in another table (TableB), you need to understand how to use subqueries and transactions effectively. This task involves ensuring data consistency while handling potential race conditions or performance issues.

#### Triage

The core question is about deleting records from one table, conditioned by the presence of corresponding records in another. The interviewer is probing your knowledge of MySQL syntax, transaction management, and possibly performance optimizations. We'll make an assumption that both tables have a common key (let's call it id).

#### First-Principles Solution
Identify Common Key: Ensure there’s a unique identifier (id) in both tables.
Subquery to Identify Matching Rows:
Use a subquery or JOIN to find rows in TableA that have matching entries in TableB.
Transaction Management: Wrap the deletion operation within a transaction to ensure atomicity and consistency.

#### Performance Considerations
Evaluate potential performance impacts, especially if dealing with large datasets.

Here’s how you can implement this:

SQL Implementation
SQL
START TRANSACTION;

-- Step 1: Identify rows in TableA that have matching entries in TableB
WITH matched_rows AS (
    SELECT id FROM TableA WHERE EXISTS (SELECT 1 FROM TableB WHERE TableB.id = TableA.id)
)

-- Step 2: Delete those identified rows from TableA
DELETE FROM TableA WHERE id IN (SELECT id FROM matched_rows);

COMMIT;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that the deletion operation is atomic and consistent. It also allows you to manage any potential issues, such as handling large datasets or ensuring data integrity.

#### Common Mistakes

Not using a transaction can lead to partial deletions if there are concurrency issues.
Ignoring performance optimizations for large tables (e.g., indexing).
Forgetting to handle failure scenarios and rollbacks in case the operation fails mid-way.

#### Example Scenario

Suppose TableA has 10,000 rows and TableB has 5,000 rows. You want to delete all rows from TableA that have a corresponding entry in TableB.

SQL
-- Sample Data (simplified)
CREATE TABLE TableA (id INT PRIMARY KEY);
INSERT INTO TableA VALUES (1), (2), (3), (4);

CREATE TABLE TableB (id INT PRIMARY KEY, data TEXT);
INSERT INTO TableB VALUES (1, 'data'), (2, 'data');

-- Running the Query
START TRANSACTION;
WITH matched_rows AS (
    SELECT id FROM TableA WHERE EXISTS (SELECT 1 FROM TableB WHERE TableB.id = TableA.id)
)
DELETE FROM TableA WHERE id IN (SELECT id FROM matched_rows);
COMMIT;

#### Performance Considerations
Indexing: Ensure that the id column is indexed in both tables to speed up the subquery.
Batch Processing: If dealing with very large datasets, consider breaking down the operation into smaller batches.

**Comparison:**

```
has
TableA
int
id
PK
TableB
int
id
PK
text
data
matching
rows
```

**Code:**

```sql
START TRANSACTION;

WITH matched_rows AS (
    SELECT id FROM TableA WHERE EXISTS (SELECT 1 FROM TableB WHERE TableB.id = TableA.id)
)
DELETE FROM TableA WHERE id IN (SELECT id FROM matched_rows);

COMMIT;
```

**Example input / output:**

```text
-- Before
SELECT * FROM TableA;
+----+
| id |
+----+
|  1 |
|  2 |
|  3 |
|  4 |
+----+

-- Running the Query

-- After
START TRANSACTION;

WITH matched_rows AS (
    SELECT id FROM TableA WHERE EXISTS (SELECT 1 FROM TableB WHERE TableB.id = TableA.id)
)
DELETE FROM TableA WHERE id IN (SELECT id FROM matched_rows);

COMMIT;

SELECT * FROM TableA;
+----+
| id |
+----+
|  3 |
|  4 |
+----+
```

**Data:**

```sql
CREATE TABLE TableA (id INT PRIMARY KEY);
INSERT INTO TableA VALUES (1), (2), (3), (4);

CREATE TABLE TableB (id INT PRIMARY KEY, data TEXT);
INSERT INTO TableB VALUES (1, 'data'), (2, 'data');
```

**Flow:**

1. Start Transaction
2. WITH matched_rows AS(SELECT id FROM TableAWHERE EXISTS (SELECT 1FROM TableB WHERE TableB.id= TableA.id))
3. DELETE FROM TableA WHEREid IN (SELECT id FROMmatched_rows)
4. Commit

