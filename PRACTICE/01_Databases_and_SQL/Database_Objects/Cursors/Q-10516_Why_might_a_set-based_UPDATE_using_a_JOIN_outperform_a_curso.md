---
type: "practice"
difficulty: "Deep"
---

# Why might a set-based UPDATE using a JOIN outperform a cursor that updates rows one at a time?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Cursors vs Set-based SQL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Cursors_vs_Set-based_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When comparing a set-based UPDATE statement with a cursor, especially in scenarios involving joins, the former can often outperform the latter due to its ability to process data more efficiently and take advantage of database optimizations. Here's why:

Set-Based Operations vs. Cursors: Set-based operations operate on multiple rows at once, whereas cursors process one row at a time. This difference in approach can lead to significant performance gains.

Database Optimizations: Modern databases are optimized for set-based operations because they can leverage indexes and other mechanisms more effectively when updating multiple rows simultaneously. In contrast, cursors may bypass these optimizations, leading to suboptimal performance.

Reduced Context Switching: Cursors require a context switch for each row processed, which can be costly in terms of CPU usage and memory management. Set-based operations avoid this overhead by processing all selected rows in one go.

Concurrency and Locking: When using cursors, you may encounter more locking conflicts as each row is updated individually. This can lead to longer transaction times and increased contention on shared resources. Set-based updates generally handle concurrency better because they involve fewer locks and transactions.

Let's delve into a practical example to understand this better.

**Code:**

```sql
-- Example of set-based UPDATE with JOIN
UPDATE T1
SET T1.column = T2.new_value
FROM table1 T1
JOIN table2 T2 ON T1.id = T2.id;

-- Example of cursor-based update (simplified pseudocode)
DECLARE cursor CURSOR FOR
SELECT T1.column, T2.new_value FROM table1 T1 JOIN table2 T2 ON T1.id = T2.id;

OPEN cursor;
FETCH NEXT FROM cursor INTO current_value, new_value;
WHILE @@FETCH_STATUS = 0 DO
    UPDATE table1 SET column = new_value WHERE id = current_id;
    FETCH NEXT FROM cursor INTO current_value, new_value;
END WHILE;
CLOSE cursor;
```

**Example input / output:**

```

The set-based UPDATE query processes 100 rows in one transaction:

PLAINTEXT
Copy
START TRANSACTION;

-- Set-based UPDATE with JOIN
UPDATE T1
SET T1.column = T2.new_value
FROM table1 T1
JOIN table2 T2 ON T1.id = T2.id;

COMMIT;

The cursor-based update process 100 rows in multiple transactions, with each transaction updating one row:

PLAINTEXT
Copy
START TRANSACTION;

-- Open the cursor and fetch the first row
DECLARE cursor CURSOR FOR
SELECT T1.column, T2.new_value FROM table1 T1 JOIN table2 T2 ON T1.id = T2.id;
OPEN cursor;
FETCH NEXT FROM cursor INTO current_value, new_value;

WHILE @@FETCH_STATUS = 0 DO
    -- Update the first row in table1 with the new value from table2
    UPDATE table1 SET column = new_value WHERE id = current_id;
    FETCH NEXT FROM cursor INTO current_value, new_value;
END WHILE;

COMMIT;
CLOSE cursor;
```

**Flow:**

1. Set-Based Update
2. Executes in one transaction
3. Processes multiple rows atonce
4. Leverages indexes andoptimizations
5. Cursors
6. Context switching per rowupdate
7. Locks and transactions foreach row
8. Much higher overhead due toindividual updates
9. Lower overhead, fasterexecution
10. Higher concurrency, betterresource utilization

