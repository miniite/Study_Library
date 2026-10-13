---
type: "practice"
difficulty: "Deep"
---

# Your endpoint sometimes returns a row that another transaction just deleted — how do you reason about whether READ COMMITTED vs REPEATABLE READ is the right fix?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with the issue of a row being read after its deletion in a MySQL transaction, we need to understand the implications of READ COMMITTED versus REPEATABLE READ. The core idea is that READ COMMITTED will allow transactions to see uncommitted changes by other transactions, whereas REPEATABLE READ ensures that each transaction sees a consistent snapshot of data as it started.

To effectively reason about which isolation level to use, we need to consider the trade-offs between consistency and performance, as well as the specific requirements of your application.

**Comparison:**

| Isolation Level | Behavior                  |
|-----------------|---------------------------|
| READ COMMITTED  | Sees uncommitted changes   |
| REPEATABLE READ | Consistent snapshot until commit |

**Code:**

```sql
-- Example SQL statements for setting transaction isolation levels

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;

SELECT * FROM my_table WHERE id = 123; -- May see recently deleted row

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;

SELECT * FROM my_table WHERE id = 123; -- Will not see the deleted row until commit
```

**Example input / output:**

```

Consider a scenario where:

A transaction starts and reads a row with id = 123.
Before this transaction commits, another transaction deletes the same row.
If the first transaction uses READ COMMITTED, it will see the deletion after the delete operation is committed.
With REPEATABLE READ, the first transaction will not see the change until its own commit.

To further illustrate:

SQL
Copy
-- Transaction 1 using REPEATABLE READ
START TRANSACTION;
SELECT * FROM my_table WHERE id = 123; -- Returns {id: 123, name: 'Example Row', deleted_at: null}
DELETE FROM my_table WHERE id = 123;   -- Deletes the row
COMMIT;

-- Transaction 2 using REPEATABLE READ
START TRANSACTION;
SELECT * FROM my_table WHERE id = 123; -- Returns nothing, as the transaction sees a consistent snapshot

-- Transaction 1 (continued)
UPDATE my_table SET deleted_at = NOW() WHERE id = 123; -- Marks row for deletion but does not delete it immediately
COMMIT;

-- Transaction 2 (continued)
SELECT * FROM my_table WHERE id = 123; -- Now sees {id: 123, name: 'Example Row', deleted_at: 'now()'}
```

**Data:**

```json
{
  "id": 123,
  "name": "Example Row",
  "deleted_at": null
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Transaction starts
6. Use READ COMMITTED?
7. Reads row, may see deletion
8. Use REPEATABLE READ?
9. Reads row, sees consistentsnapshot until commit
10. Use other isolation level?
11. Reads row, may not see recentchanges

