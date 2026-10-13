---
type: "practice"
difficulty: "Basic"
---

# What does the ORDER BY clause do, and how do you sort by multiple columns with mixed ASC and DESC?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The ORDER BY clause in PostgreSQL is used to sort the result set of a query based on one or more columns. When sorting by multiple columns, you can specify different order directions (ASC for ascending and DESC for descending) for each column.

#### Triage
This question is probing your understanding of SQL's ORDER BY clause, specifically how it works with multiple columns and mixed sort orders. We'll break down the functionality from first principles to ensure a clear understanding.

When you use ORDER BY, PostgreSQL sorts the rows in ascending order by default (which corresponds to ASC). If you want to sort by multiple columns, you can list them after the clause, separated by commas. The order of the columns and the keywords (ASC or DESC) determine the sorting logic. By default, all columns are sorted in ascending order unless specified otherwise.

#### First-Principles Solution

Single Column Sorting:

For a single column, you can sort as follows:
SQL
SELECT * FROM table ORDER BY column ASC;
This will return the rows ordered by column in ascending order.
To sort in descending order, use DESC:
SQL
SELECT * FROM table ORDER BY column DESC;

Multiple Columns Sorting:

If you want to sort by multiple columns with mixed ASC and DESC directions, list the columns accordingly. For example, if you have a title and date column, and you want titles in ascending order but dates in descending order, use:
SQL
SELECT * FROM table ORDER BY title ASC, date DESC;
This will first sort by title in ascending order and then, for rows with the same title, it will sort by date in descending order.

Handling NULL Values:

By default, PostgreSQL treats NULL values as the lowest possible value when sorting in ASCENDING order and highest possible value when sorting in DESCENDING order.
If you want to handle NULLs differently (e.g., sort them after non-NULL values), use NULLS FIRST or NULLS LAST:
SQL
SELECT * FROM table ORDER BY column DESC NULLS LAST;

Descending Order for All Columns:

If you want to reverse the order of all columns, you can add a - sign before the column name in your query (though it's more common to use DESC):
SQL
SELECT * FROM table ORDER BY -column1, -column2;
This is equivalent to:
SQL
SELECT * FROM table ORDER BY column1 DESC, column2 DESC;

#### Common Mistakes

Not specifying the direction (ASC or DESC) for all columns can lead to unexpected results.
Assuming that NULLS FIRST works by default when it needs to be explicitly specified.

#### Closing Summary
Understanding and effectively using the ORDER BY clause in PostgreSQL is crucial for data retrieval. By mastering how to sort based on multiple columns with mixed directions, you ensure that your queries return the desired results efficiently.

#### Common Mistakes/Follow-Ups

Always specify the direction (ASC or DESC) explicitly.
Be aware of how NULL values are handled and use NULLS FIRST/NULLS LAST if needed.

#### Cons
ider performance implications when dealing with large datasets, as sorting can be a costly operation.

**Code:**

```sql
-- Example 1: Single column ASCENDING sort
SELECT * FROM employees ORDER BY last_name;

-- Example 2: Multiple columns with mixed order
SELECT * FROM orders ORDER BY customer_name ASC, order_date DESC;
```

**Flow:**

1. ASC
2. DESC
3. ASC
4. DESC
5. ASC
6. DESC
7. ASC
8. DESC
9. Single Column Sort
10. Sort Direction?
11. Ascending Order
12. Descending Order
13. Final Query
14. Final Query
15. Multiple Columns Sort
16. Column 1 Sort Direction?
17. I
18. J
19. Column 2 Sort Direction?
20. Column 2 Sort Direction?
21. M
22. N
23. O
24. P
25. Final Query
26. Final Query
27. Final Query
28. Final Query

