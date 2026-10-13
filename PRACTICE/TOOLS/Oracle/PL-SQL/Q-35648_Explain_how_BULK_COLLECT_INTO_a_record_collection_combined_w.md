---
type: "practice"
difficulty: "Expert"
---

# Explain how BULK COLLECT INTO a record collection combined with FORALL INSERT differs from row-by-row inserts in redo generation and latch behavior.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, BULK COLLECT INTO is used to fetch multiple rows into an associative array or a table collection at once, whereas FORALL INSERT is employed to perform bulk DML operations by iterating over elements of a collection and inserting them row-by-row. This question delves into how these techniques differ in terms of redo generation and latch behavior.

#### Triaging the Question

The core of this question lies in understanding the performance implications of using BULK COLLECT INTO combined with FORALL INSERT versus traditional row-by-row inserts in Oracle PL/SQL. Specifically, we need to explore their impact on:

Redo Generation: The amount and nature of redo logs generated during these operations.
Latch Behavior: How the different methods interact with shared memory structures.

#### Assumptions
We are working within an Oracle database environment.
The table being inserted into has a non-null primary key, which is a common use case for FORALL INSERT.

#### First-Principles Solution
1. Redo Generation

Row-by-row Inserts: When you perform row-by-row inserts, each insert statement generates its own redo record. Each insert updates the table block in memory, and this change is recorded as a redo entry. This can lead to a high volume of redo entries, especially for large datasets.

SQL
FOR i IN 1..records.count LOOP
    INSERT INTO my_table (column1, column2) VALUES (records(i).col1, records(i).col2);
END LOOP;

BULK COLLECT INTO + FORALL INSERT: Using BULK COLLECT INTO combined with FORALL INSERT, you first collect multiple rows into a collection and then insert all of them in one go. This results in fewer redo entries because the entire collection is processed as a single transaction, reducing the number of individual DML operations.

SQL
DECLARE
    TYPE my_rec_type IS RECORD (col1 NUMBER, col2 VARCHAR2(100));
    TYPE rec_tab_type IS TABLE OF my_rec_type;
BEGIN
    -- BULK COLLECT INTO: Collect multiple rows into a collection.
    FORALL i IN 1..records.count
        INSERT INTO my_table (column1, column2) VALUES (records(i).col1, records(i).col2);
END;
2. Latch Behavior

Row-by-row Inserts: Each insert statement may require latches to update the table blocks in memory. This can lead to contention and increased latch activity, especially if the inserts are frequent or if there is a large number of concurrent transactions.

SQL
-- Example of row-by-row insert causing latch contention.
FOR i IN 1..records.count LOOP
    INSERT INTO my_table (column1, column2) VALUES (records(i).col1, records(i).col2);
END LOOP;

BULK COLLECT INTO + FORALL INSERT: Using BULK COLLECT INTO combined with FORALL INSERT, the entire collection is processed in a single transaction. This reduces the number of times latches need to be acquired for block updates, thereby reducing latch contention and improving performance.

#### Closing Summary

In summary, using BULK COLLECT INTO combined with FORALL INSERT can significantly reduce redo generation and minimize latch contention compared to row-by-row inserts. This makes it a more efficient approach in scenarios where bulk data manipulation is required. However, it's crucial to consider the specific use case and database environment, as the trade-offs between performance benefits and potential issues (like increased memory usage) must be carefully balanced.

#### Common Mistakes/Follow-Ups
Misunderstanding Latch Contention: Not recognizing that latch contention can increase with row-by-row inserts.
Overlooking Redo Logs: Assuming that more redo logs are always better, without considering the overhead they introduce.
Not Considering Transaction Management: Forgetting to manage transactions properly when using FORALL INSERT, which can lead to data inconsistencies if not handled carefully.

**Comparison:**

| Column | Data Type |
|--------|-----------|
| id     | NUMBER    |
| value  | VARCHAR2  |

**Code:**

```sql
-- Example of row-by-row insert causing latch contention.
FOR i IN 1..records.count LOOP
    INSERT INTO my_table (column1, column2) VALUES (records(i).col1, records(i).col2);
END LOOP;

-- BULK COLLECT INTO + FORALL INSERT example.
DECLARE
    TYPE my_rec_type IS RECORD (col1 NUMBER, col2 VARCHAR2(100));
    TYPE rec_tab_type IS TABLE OF my_rec_type;
BEGIN
    -- BULK COLLECT INTO: Collect multiple rows into a collection.
    my_collection.BulkCollectInto(records);

    -- FORALL INSERT example.
    FORALL i IN 1..records.count
        INSERT INTO my_table (column1, column2) VALUES (records(i).col1, records(i).col2);
END;
```

**Example input / output:**

```text
-- Input for BULK COLLECT INTO and FORALL INSERT.
records := rec_tab_type(
    (col1 => 1, col2 => 'one'),
    (col1 => 2, col2 => 'two'),
    (col1 => 3, col2 => 'three')
);

-- Output after the insert operation using BULK COLLECT INTO and FORALL INSERT.
my_table := (
    (id => 1, value => 'one'),
    (id => 2, value => 'two'),
    (id => 3, value => 'three')
);
```

**Data:**

```plaintext
-- Sample data for the records collection.
records := rec_tab_type(
    (col1 => 1, col2 => 'one'),
    (col1 => 2, col2 => 'two'),
    (col1 => 3, col2 => 'three')
);
```

**Flow:**

1. Start
2. Row-by-row Insert
3. Generate redo for each row
4. Latch contention per insert
5. BULK COLLECT INTO + FORALLINSERT
6. Generate single bulk redoentry
7. Reduce latch contention bybatch processing

