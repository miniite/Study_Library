---
type: "practice"
difficulty: "Deep"
---

# Why can NOT IN with a subquery return zero rows unexpectedly when NULLs are involved?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ WHERE vs HAVING](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/WHERE_vs_HAVING.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with databases, NOT IN queries combined with subqueries can sometimes behave unexpectedly due to the presence of NULL values. This behavior is often counterintuitive and can lead to unexpected results where the query returns zero rows when it should return some. Let's explore why this happens from first principles.

Firstly, a brief overview: NOT IN checks whether a value does not exist in the result set of a subquery. If there are NULL values involved, the behavior can be different because SQL treats NULL as unknown. This means that NULL NOT IN (x1, x2) is considered false even if x1 and x2 are non-NULL. Now, let's dive into why this happens.

**Comparison:**

| example_table | another_table |
|---------------|---------------|
| id | value | id | value |
| --- | ---- | --- | ---- |
| 1 | NULL | 1 | a    |
| 2 | a    |       |      |
| 3 | b    |       |      |
| id | value |
|----|-------|
| 3  | b     |

**Code:**

```sql
-- Example table schema
CREATE TABLE example_table (
    id INT,
    value VARCHAR(50)
);
```

**Example input / output:**

```
Worked Input->Output

Input:

example_table:

PLAINTEXT
Copy
id | value
---|------
 1 | NULL
 2 | 'a'
 3 | 'b'

another_table:

PLAINTEXT
Copy
id | value
---|------
 1 | 'a'

Output:

PLAINTEXT
Copy
id | value
---|------
  3 | 'b'
Explanation

The query using LEFT JOIN returns the rows from example_table that do not have a match in another_table. In this case, only row with id 3 ('b') is returned because it does not appear in another_table.
```

**Data:**

```

INSERT INTO example_table (id, value) VALUES (1, NULL), (2, 'a'), (3, 'b');

CODE
Copy

### Triaging the Question

The question is probing your understanding of how SQL handles `NULL` values and the behavior of set operations like `NOT IN`. The key assumptions are that you're familiar with basic SQL syntax and understand the special treatment given to `NULL`.

### First-Principles Solution

1. **Understanding `NULL`:** In SQL, `NULL` represents an unknown or undefined value. It is not a zero or empty string; it's more akin to a placeholder for "I don't know."

2. **Behavior of `NOT IN`:** The `NOT IN` condition checks whether a value does not exist in the result set of a subquery. For example, `1 NOT IN (NULL)` evaluates to false because SQL treats `NULL` as an unknown value.

3. **Subquery and `NULL`:** When you use a subquery with `NULL`, the subquery itself doesn't return any specific row; it essentially returns an empty set, but with `NULL`s included in that empty set.

4. **Example Scenario:**
   ```sql
   SELECT * FROM example_table WHERE value NOT IN (SELECT value FROM another_table);

If another_table contains NULL, the subquery will return a result set containing NULL. The outer query then evaluates value NOT IN (subquery_result) for each row. Since NULL is an unknown, 1 NOT IN ('a', 'b', NULL) and 3 NOT IN ('a', 'b', NULL) both evaluate to false because SQL considers NULL NOT IN (x1, x2) as false.

Expected vs. Actual Result:
Expected result: Rows with values not in the subquery.
Actual result: No rows returned due to the presence of NULL in the subquery.
Closing Summary and Common Mistakes/Follow-Ups

In summary, the behavior of NOT IN with NULL is due to SQL's treatment of NULL as an unknown value. This can lead to unexpected results when used with subqueries that return NULL. To avoid such issues, consider using LEFT JOIN or other set operations like EXISTS.

Common mistakes include assuming that NULL NOT IN (subquery) will behave the same way as non-NULL values.

Follow-ups could involve discussing alternative query structures to handle NULL values effectively.

SQL
Copy
-- Example using LEFT JOIN instead of NOT IN with NULL
SELECT * FROM example_table t1
LEFT JOIN another_table t2 ON t1.value = t2.value
WHERE t2.value IS NULL;
PLAINTEXT
Copy
-- This query would return the rows from `example_table` that do not have a match in `another_table`.
```

**Flow:**

1. Query using NOT IN
2. Is subquery NULL?
3. Return no rows
4. Continue evaluation
5. Check each row againstsubquery result
6. Row with value NOT IN(subquery) is returned

