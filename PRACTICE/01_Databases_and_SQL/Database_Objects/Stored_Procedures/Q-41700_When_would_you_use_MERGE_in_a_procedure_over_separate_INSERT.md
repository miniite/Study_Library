---
type: "practice"
difficulty: "Deep"
---

# When would you use MERGE in a procedure over separate INSERT/UPDATE branches, and what performance pitfalls does MERGE have?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Stored Procedure Performance and Design](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between using MERGE in a stored procedure versus separate INSERT and UPDATE branches, you need to weigh the trade-offs of transactional integrity, simplicity, and performance. MERGE is particularly useful when you want to handle both insertions and updates based on the existence of a key in one atomic operation. However, it comes with its own set of performance pitfalls that can significantly impact your database's efficiency.

To understand this better, let’s break down the core concepts and then dive into the details.

**Comparison:**

| ID  | Name       |
|-----|------------|
| 123 | John Doe   |
| 456 | Jim Smith  |

**Code:**

```sql
-- Example MERGE statement
MERGE INTO Customers AS target
USING (SELECT '123' AS ID, 'John Doe' AS Name) AS source
ON target.ID = source.ID
WHEN MATCHED THEN
    UPDATE SET target.Name = source.Name
WHEN NOT MATCHED THEN
    INSERT VALUES (source.ID, source.Name);
```

**Example input / output:**

```text
Given the initial data and a `MERGE` statement as above, if the source contains an ID that exists in the target table (e.g., `'123'`), then it will update the name. If not (`'456'`), it inserts a new record.
```

**Data:**

```json
{
  "Customers": [
    { "ID": "123", "Name": "Jane Doe" },
    { "ID": "456", "Name": "Jim Smith" }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there a matching record?
5. Update the existing record
6. Insert a new record
7. End Transaction
8. End Transaction

