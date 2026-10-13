---
type: "practice"
difficulty: "Basic"
---

# How would you remove duplicate rows during a transformation step in SQL?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Source Changes and Spikes](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To remove duplicate rows during an ETL (Extract, Transform, Load) or ELT (Extract, Load, Transform) process using SQL, we can leverage built-in functions and clauses designed to handle duplicates effectively. The core idea is to use the DISTINCT keyword for simple scenarios, while more complex cases might require additional logic such as window functions.

We'll first briefly summarize the approach: identifying unique rows based on specific columns or a combination of them, then ensuring data integrity by either dropping duplicates directly in the ETL process or marking them for further processing. The goal is to ensure that each row is processed only once, which can significantly improve performance and reduce redundancy.

Now, let's dive into the details:

**Code:**

```sql
-- Example of removing duplicates using DISTINCT
SELECT DISTINCT column1, column2, column3 FROM source_table;

-- Or with a common table expression (CTE) for more complex scenarios
WITH filtered_data AS (
    SELECT column1, column2, column3, ROW_NUMBER() OVER (PARTITION BY column1, column2, column3 ORDER BY id) as row_num
    FROM source_table
)
SELECT * FROM filtered_data WHERE row_num = 1;
```

**Example input / output:**

```text
-- Example input data
source_table:
id | column1 | column2 | column3
---------------------------
1  | A       | B       | C
2  | A       | B       | C
3  | D       | E       | F

-- After applying the distinct or window function approach, we get:
filtered_data:
id | column1 | column2 | column3
---------------------------
1  | A       | B       | C
3  | D       | E       | F
```

**Flow:**

1. Yes
2. No
3. Start
4. Is data simple?
5. Use DISTINCT
6. Use window functions
7. Apply transformation
8. Apply window function logic
9. Output unique rows
10. End

