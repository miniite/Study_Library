---
type: "practice"
difficulty: "Intermediate"
---

# What does a MERGE statement do, and why is it central to an ELT upsert step in a warehouse pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Large Batch Updates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The MERGE statement in SQL is a powerful tool used to perform data integration tasks, particularly when you need to upsert (insert or update) rows based on matching conditions. In an ELT (Extract, Load, Transform) pipeline, the MERGE statement is central because it efficiently handles merging new and updated records into a target database without requiring complex temporary tables or manual comparisons.

The core idea of a MERGE statement is to match source data with existing data in a target table based on a specified key. It then performs one of three actions: insert, update, or do nothing if there's no match (a delete operation can also be included). This capability makes it indispensable for maintaining an up-to-date and accurate database.

Let’s break down the MERGE statement step by step from first principles.

**Comparison:**

```
Existing Table	New Data	Result
id	name	age
1	Alicia	25
2	null	32
(no record)	Charlie	40
```

**Code:**

```sql
-- Example MERGE Statement Syntax

WITH source_data AS (
    SELECT id, name, age
    FROM new_data
),
target_table AS (
    SELECT id, name, age
    FROM existing_table
)

MERGE INTO target_table AS tgt
USING source_data AS src
ON tgt.id = src.id  -- Match condition

WHEN MATCHED THEN
    UPDATE SET tgt.name = src.name, tgt.age = src.age

WHEN NOT MATCHED THEN
    INSERT (id, name, age) VALUES (src.id, src.name, src.age);
```

**Example input / output:**

```

Given the new_data and existing_table, here’s a dry run of the MERGE statement:

id = 1: Matched; update name from "Alicia" to "Alice", age from 25 to 28.
id = 2: Matched; update name from null to "Bob", age from 32 to 34.
id = 3: Not matched; insert new record (3, "Charlie", 40).

After execution:

JSON
Copy
{
  "result": {
    "id": [1, 2, 3],
    "name": ["Alice", "Bob", "Charlie"],
    "age": [28, 34, 40]
  }
}
```

**Data:**

```json
{
  "new_data": {
    "id": [1, 2, 3],
    "name": ["Alice", "Bob", "Charlie"],
    "age": [28, 34, 40]
  },
  "existing_table": {
    "id": [1, 2],
    "name": ["Alicia", null],
    "age": [25, 32]
  }
}
```

**Flow:**

1. Yes
2. Matched
3. Not matched
4. Start
5. Is the source data available?
6. Merge into target table
7. Identify matching conditions
8. Check if matched
9. Update existing records
10. Insert new records
11. End
12. End

