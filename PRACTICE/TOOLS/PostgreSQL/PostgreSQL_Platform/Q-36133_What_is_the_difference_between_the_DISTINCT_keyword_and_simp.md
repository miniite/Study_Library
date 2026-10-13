---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the DISTINCT keyword and simply removing duplicates in application code?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PostgreSQL Query Basics](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/PostgreSQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the difference between using DISTINCT in PostgreSQL queries versus manually removing duplicates in application code, let's first break down each approach.

TL;DR
PostgreSQL's DISTINCT: A SQL keyword that filters out duplicate rows directly in the database query.
Removing Duplicates in Application Code: Logic implemented outside of the database, which can be more complex and less efficient compared to using DISTINCT.

#### Triaging the Question

The core of this question is about understanding when and why you would choose one method over the other. The interviewer is probing your knowledge of SQL, understanding of performance considerations, and ability to think through trade-offs.

#### First Principles Solution
Using DISTINCT in PostgreSQL Queries

SQL Syntax:

SQL
SELECT DISTINCT column_name FROM table_name;

Internals: When you use the DISTINCT keyword, PostgreSQL internally identifies duplicate rows based on the specified columns and removes them before returning the results.

Advantages:

Performance: Generally, PostgreSQL's query optimizer can handle DISTINCT more efficiently than application code.
Scalability: The database handles duplicates at a lower level, which is often optimized for large datasets.

#### Cons
istency: Ensures data consistency directly in the database.

Edge Cases:

Null Values: If you have NULL values and want to treat them as unique, use DISTINCT ON or COALESCE.
Multiple Columns: Use multiple columns within the DISTINCT clause for more complex scenarios.
Removing Duplicates in Application Code

Logic Implementation:

PYTHON
result = []
seen = set()
for row in query_result:
    if row not in seen:
        result.append(row)
        seen.add(row)

Advantages:

Flexibility: You can implement logic that PostgreSQL cannot handle, such as complex conditions or transformations.
Customization: More control over the data manipulation process.

Disadvantages:

Performance Overhead: Moving data between the database and application code increases latency and computation time.
Scalability Issues: As datasets grow larger, this approach can become less efficient compared to native SQL operations.

#### Closing Summary

In summary, DISTINCT in PostgreSQL is generally more performant and scalable for removing duplicates. It leverages the database's optimization capabilities, ensuring consistent behavior across all queries. Application-level logic might be necessary for complex transformations or when specific conditions need to be applied, but it should be used judiciously due to potential performance drawbacks.

#### Common Mistakes/Follow-ups
Overusing DISTINCT: Using DISTINCT unnecessarily can slow down queries.
Ignoring Indexes: Ensure proper indexing on columns used with DISTINCT.
Complex Conditions: Applying complex conditions in application code instead of SQL might lead to performance issues.

Do you have any specific use cases where you would prefer to handle duplicates in the application code, or are there any additional questions I can help clarify?

**Comparison:**

```
Value
A
B
C
```

**Code:**

```

-- PostgreSQL DISTINCT Example SELECT DISTINCT column_name FROM table_name;

CODE
Copy
```python
# Python Example of Removing Duplicates in Application Code
result = []
seen = set()
for row in query_result:
    if row not in seen:
        result.append(row)
        seen.add(row)
```

**Example input / output:**

```

Input (PostgreSQL): SELECT DISTINCT value FROM table_name;

Output: value

A B C

CODE
Copy

```python
# Input Data
query_result = [
    {"id": 1, "value": "A"},
    {"id": 2, "value": "B"},
    {"id": 3, "value": "C"},
    {"id": 4, "value": "A"}
]

# Output after removing duplicates in application code
result = []
seen = set()
for row in query_result:
    if row not in seen:
        result.append(row)
        seen.add(row)

print(result)
PLAINTEXT
Copy
[{'id': 1, 'value': 'A'}, {'id': 2, 'value': 'B'}, {'id': 3, 'value': 'C'}]
```

**Data:**

```json
{
  "query_result": [
    {"id": 1, "value": "A"},
    {"id": 2, "value": "B"},
    {"id": 3, "value": "C"},
    {"id": 4, "value": "A"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query with DISTINCT
4. Is there a complex condition?
5. Implement logic in applicationcode
6. Use DISTINCT in SQL
7. Handle complex conditionsand transformations
8. Efficient query execution bythe database

