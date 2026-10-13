---
type: "practice"
difficulty: "Deep"
---

# What is a correlated aggregate and how would you compute a running total using a window function?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Window Functions Overview](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Window_Functions_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A correlated aggregate is a type of SQL query where an aggregate function (like SUM, AVG) is used within a subquery that references the current row in the outer query. This means the aggregate calculation is dependent on each row's context from the main table or another table.

To compute a running total using window functions, we leverage SQL’s ability to perform calculations over a set of rows related to the current row. We will use the SUM() function with a OVER clause and a PARTITION BY if necessary, to calculate the cumulative sum up to each row in our result.

First, let's dive into the core concept before we get to an example.

**Code:**

```sql
-- Example query using window functions to compute a running total
SELECT
    id,
    amount,
    SUM(amount) OVER (ORDER BY id) AS running_total
FROM transactions;
```

**Example input / output:**

```

For the sample data, if we run the query:

SQL
Copy
SELECT
    id,
    amount,
    SUM(amount) OVER (ORDER BY id) AS running_total
FROM transactions;

We get the following output:

MARKDOWN
Copy
| id |  amount | running_total |
|----|--------:|--------------:|
| 1  |   100  |             100 |
| 2  |   200  |             300 |
| 3  |   -50  |             250 |
| 4  |   150  |             400 |
```

**Data:**

```

The transactions table might look like this:

YAML
Copy
id  |  amount
----|--------
1   |  100
2   |  200
3   |  -50
4   |  150
```

