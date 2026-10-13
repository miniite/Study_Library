---
type: "practice"
difficulty: "Basic"
---

# What is the LIKE operator and how does it differ from ILIKE?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Full-text Search in Postgres](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Full-text_Search_in_Postgres.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The LIKE operator in PostgreSQL is used to match a pattern against a string, which can be very powerful but also requires careful use due to its performance implications. It differs from ILIKE, which performs case-insensitive matching. Let's break down the differences and how they work.

#### Triaging the Question
Understanding LIKE: The LIKE operator allows you to search for a pattern in a column value, using wildcards such as % (any number of characters) and _ (a single character).
Case Sensitivity: By default, LIKE is case-sensitive.

#### Performance Considerations
Using LIKE with the % or _ wildcard at the beginning can lead to full table scans, which can be inefficient.
How LIKE Works

The basic syntax for using LIKE in a query looks like this:

SQL
SELECT column_name FROM table_name WHERE column_name LIKE 'pattern';

For example, if you want to find all records where the name starts with "Jo", you would use:

SQL
SELECT * FROM employees WHERE first_name LIKE 'Jo%';
Case Sensitivity

By default, LIKE is case-sensitive. This means that searching for a specific letter in a column will only return results that match exactly, including capitalization.

Introducing ILIKE

The ILIKE operator works similarly to LIKE, but it performs a case-insensitive search. For example:

SQL
SELECT * FROM employees WHERE first_name ILIKE 'jo%';

This query would return any names starting with "Jo" or "jo", regardless of the capitalization.

Practical Example

Let's consider an example schema and some sample data to illustrate this.

YAML
# Sample Employees Table Schema
employees:
  - id: int
  - first_name: varchar(50)
  - last_name: varchar(50)

# Sample Data
employees_data:
  - {id: 1, first_name: 'John', last_name: 'Doe'}
  - {id: 2, first_name: 'Joan', last_name: 'Smith'}
  - {id: 3, first_name: 'Joe', last_name: 'Johnson'}
  - {id: 4, first_name: 'joseph', last_name: 'Williams'}
Running the Queries

To see the difference between LIKE and ILIKE, we can run the following queries:

SQL
-- Using LIKE with case sensitivity
SELECT * FROM employees WHERE first_name LIKE 'jo%';

-- Using ILIKE for case-insensitive search
SELECT * FROM employees WHERE first_name ILIKE 'jo%';

#### Common Mistakes
and Follow-Ups
Full Table Scan: When using LIKE with % or _ at the beginning, PostgreSQL may perform a full table scan instead of using an index.
Performance Optimization: Always consider whether you can replace LIKE with a more efficient query or use a specific index to improve performance.
Case Sensitivity in Full-Text Search: If case sensitivity is important and you need to search for terms like "Jo" and "jo," you might want to avoid using ILIKE.

#### Closing Summary

In summary, the LIKE operator allows pattern matching in PostgreSQL with full case sensitivity by default. The ILIKE operator provides a similar functionality but with case insensitivity. Understanding these operators and their differences is crucial for effective database querying.

**Comparison:**

| ID | First Name | Last Name |
|----|------------|-----------|
| 1  | John       | Doe       |
| 2  | Joan       | Smith     |
| 3  | Joe        | Johnson   |
| 4  | joseph     | Williams  |

**Code:**

```sql
-- Example queries
SELECT * FROM employees WHERE first_name LIKE 'jo%';
SELECT * FROM employees WHERE first_name ILIKE 'jo%';
```

**Example input / output:**

```text
-- Running the query with LIKE and ILIKE
LIKE_query = "SELECT * FROM employees WHERE first_name LIKE 'jo%';"
ILIKE_query = "SELECT * FROM employees WHERE first_name ILIKE 'jo%';"

# Sample output for LIKE query
EXPECTED_OUTPUT_LIKE = [
    {'id': 1, 'first_name': 'John', 'last_name': 'Doe'},
    {'id': 2, 'first_name': 'Joan', 'last_name': 'Smith'}
]

# Sample output for ILIKE query
EXPECTED_OUTPUT_ILIKE = [
    {'id': 1, 'first_name': 'John', 'last_name': 'Doe'},
    {'id': 2, 'first_name': 'Joan', 'last_name': 'Smith'},
    {'id': 3, 'first_name': 'Joe', 'last_name': 'Johnson'},
    {'id': 4, 'first_name': 'joseph', 'last_name': 'Williams'}
]
```

**Data:**

```
Sample Employees Table Schema

employees:

id: int
first_name: varchar(50)
last_name: varchar(50)
Sample Data

employees_data:

{id: 1, first_name: 'John', last_name: 'Doe'}
{id: 2, first_name: 'Joan', last_name: 'Smith'}
{id: 3, first_name: 'Joe', last_name: 'Johnson'}
{id: 4, first_name: 'joseph', last_name: 'Williams'}
CODE
Copy

### Running the Queries
To see the difference between `LIKE` and `ILIKE`, we can run the following queries:
```sql
-- Using LIKE with case sensitivity
SELECT * FROM employees WHERE first_name LIKE 'jo%';

-- Using ILIKE for case-insensitive search
SELECT * FROM employees WHERE first_name ILIKE 'jo%';
Common Mistakes and Follow-Ups
Full Table Scan: When using LIKE with % or _ at the beginning, PostgreSQL may perform a full table scan instead of using an index.
Performance Optimization: Always consider whether you can replace LIKE with a more efficient query or use a specific index to improve performance.
Case Sensitivity in Full-Text Search: If case sensitivity is important and you need to search for terms like "Jo" and "jo," you might want to avoid using ILIKE.
Closing Summary

In summary, the LIKE operator allows pattern matching in PostgreSQL with full case sensitivity by default. The ILIKE operator provides a similar functionality but with case insensitivity. Understanding these operators and their differences is crucial for effective database querying.

SQL
Copy
-- Example queries
SELECT * FROM employees WHERE first_name LIKE 'jo%';
SELECT * FROM employees WHERE first_name ILIKE 'jo%';
YAML
Copy
employees_data:
  - {id: 1, first_name: 'John', last_name: 'Doe'}
  - {id: 2, first_name: 'Joan', last_name: 'Smith'}
  - {id: 3, first_name: 'Joe', last_name: 'Johnson'}
  - {id: 4, first_name: 'joseph', last_name: 'Williams'}
```

**Flow:**

1. Yes
2. No
3. LIKE Operator
4. Case Sensitivity?
5. Use LIKE with Care
6. Use ILIKE for Case Insensitivity
7. Consider Indexes andPerformance
8. Ensure Full MatchingCapabilities

