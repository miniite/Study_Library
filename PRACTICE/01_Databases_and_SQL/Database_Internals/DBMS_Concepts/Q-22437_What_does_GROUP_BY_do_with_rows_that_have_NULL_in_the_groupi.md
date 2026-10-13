---
type: "practice"
difficulty: "Intermediate"
---

# What does GROUP BY do with rows that have NULL in the grouping column?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ WHERE vs HAVING](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GROUP BY is a fundamental SQL operation used to aggregate data based on one or more columns. When you include a column with NULL values in your GROUP BY clause, the behavior can be subtle and might lead to unexpected results if not understood properly.

#### Triage

The question asks about how rows with NULL values behave when included in the GROUP BY clause. The key here is understanding that SQL's treatment of NULL as a special value that doesn't equal anything, including itself, can affect grouping operations.

#### First-Principles Solution

In SQL, NULL represents an unknown or missing value. When you use a column with NULL values in a GROUP BY clause, each NULL is treated as a distinct group by default. This means that rows with the same non-null value are grouped together, while those with NULL form their own separate groups.

For example:

If we have data like this:
OrderID	Product
101	Apple
102	NULL
103	Orange
104	NULL

And you run the query:

SQL
SELECT Product, COUNT(*)
FROM Orders
GROUP BY Product;

The output will be:

Product	COUNT(*)
Apple	1
NULL	2
Orange	1

Here, NULL is treated as a distinct group and forms its own row in the result set.

#### Closing Summary

To summarize, GROUP BY treats NULL as a special case. Each NULL value creates its own separate group, even if there are multiple rows with the same NULL value. This behavior can be useful for certain types of analysis but might lead to unexpected results in other scenarios.

#### Common Mistakes
and Follow-Ups:
Misunderstanding NULL: Some developers might mistakenly assume that NULL values in a column will form one group, leading to incorrect expectations about the query output.

#### Performance Considerations
Understanding this behavior can help in optimizing queries. For instance, if you're grouping on a column with many NULL values, it could impact performance due to additional storage and processing overhead for those groups.

Do you have any follow-up questions or scenarios where you'd like to apply this concept?

**Code:**

```sql
-- Example SQL query
SELECT Product, COUNT(*)
FROM Orders
GROUP BY Product;
```

**Example input / output:**

```text
-- Query Result:
Product | COUNT(*)
--------|---------
Apple   | 1
NULL    | 2
Orange  | 1
```

**Data:**

```json
[
  {"OrderID": 101, "Product": "Apple"},
  {"OrderID": 102, "Product": null},
  {"OrderID": 103, "Product": "Orange"},
  {"OrderID": 104, "Product": null}
]
```

