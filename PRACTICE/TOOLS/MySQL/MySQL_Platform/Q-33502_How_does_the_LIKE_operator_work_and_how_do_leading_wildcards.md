---
type: "practice"
difficulty: "Intermediate"
---

# How does the LIKE operator work and how do leading wildcards affect index usage?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Full-text Search](<../../../../TOOLS/MySQL/MySQL_Platform/Full-text_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this intermediate-level interview question, we'll dive into understanding how MySQL's LIKE operator works and explore its interaction with indexes, particularly focusing on leading wildcards. Let's begin by summarizing the key points:

The LIKE operator is commonly used in SQL to search for a pattern within a column.
Leading wildcards (% at the start of the pattern) can significantly impact index usage.

Now, let's break down the question step-by-step from first principles.

#### Triage
What is Being Probed

The interviewer is likely interested in your understanding of SQL query performance and how different patterns affect indexing strategies. They want to see if you can explain both the theory behind LIKE and its practical implications on query execution plans.

#### Assumptions
and Key Concepts
Index Usage: We will assume that an appropriate index exists for the column being queried.
Execution Plan: Understanding of how MySQL generates and uses execution plans.
Step-by-Step Solution
1. Basic Operation of LIKE

The LIKE operator in SQL allows searching for a pattern within strings. It supports two wildcards:

%: Matches any sequence of characters (including an empty string).
_: Matches exactly one character.

For example, the query SELECT * FROM users WHERE name LIKE 'joh%' would return all names starting with "joh".

2. Index Scanning and Full Table Scan

When using LIKE, MySQL has to decide whether it can use an index efficiently or if a full table scan is necessary.

Index Scan: If the pattern starts with a non-wildcard character, like 'joh%', MySQL might be able to perform an efficient index scan.
Full Table Scan: If the pattern starts with a wildcard ('%o'), MySQL cannot use the index effectively and will likely opt for a full table scan.
3. Impact of Leading Wildcards

Leading wildcards are problematic because they prevent MySQL from using the index efficiently:

SQL
SELECT * FROM users WHERE name LIKE '%john'
Inefficient Index Usage: The LIKE '%john' pattern forces MySQL to perform a table scan, as it can't use the index to quickly narrow down the search.
4. Performance Considerations and Workarounds

To mitigate performance issues:

Avoid Leading Wildcards: Use non-wildcard patterns when possible.
Full-Text Indexes: For more complex searches involving leading wildcards, consider using full-text indexes (though these are not available in all situations).
Covering Indexes: Ensure that the index includes other frequently queried columns to improve overall query performance.

#### Closing Summary
and Common Mistakes

Understanding how LIKE works with indexes is crucial for writing efficient SQL queries. Common mistakes include:

Using leading wildcards without realizing their impact on performance.
Failing to recognize when a full table scan is necessary versus an index scan.

In practice, you should always optimize your queries by minimizing the use of leading wildcards and considering other indexing strategies where appropriate.

**Code:**

```sql
-- Example of inefficient query due to leading wildcard
SELECT * FROM users WHERE name LIKE '%john';

-- Efficient query using non-wildcard pattern
SELECT * FROM users WHERE name = 'john';
```

**Flow:**

1. No
2. Yes
3. Query with leading wildcard
4. Is index used?
5. Full table scan
6. Index scan, but only if patternis not at the start
7. Inefficient performance
8. Efficient performance

