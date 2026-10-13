---
type: "practice"
difficulty: "Deep"
---

# What is a prefix index and when would you use one on a long VARCHAR column?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Full-text Search](<../../../../TOOLS/MySQL/MySQL_Platform/Full-text_Search.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A prefix index is an optimization technique in MySQL where only the first part of a string value is indexed, rather than the entire length. This can significantly reduce storage space and improve query performance when searching on partial values. In this context, you would use a prefix index on a long VARCHAR column to optimize queries that search for substrings at the beginning of the text.

#### Triage

The question asks about prefix indexes in MySQL, specifically in relation to long VARCHAR columns. The interviewer is probing your understanding of indexing techniques and when they are beneficial. Key assumptions include:

Familiarity with MySQL's indexing mechanisms.
Understanding of how different types of queries benefit from varying index strategies.
Step-by-Step Solution

Understanding the Problem:

Prefix indexes target situations where searches frequently involve querying for substrings at the beginning of a VARCHAR column, such as filtering by the first few characters of a string.

Why Use a Prefix Index?

Reduced Storage: Storing only the prefix (e.g., the first 10 characters) requires less disk space compared to storing the entire value.
Faster Query Execution: Shorter indexes mean faster index scans and lookups, especially when combined with LIKE queries that use the leading wildcard (%).

#### Implementation
Considerations:

Define a prefix length based on typical query patterns. For example, if 90% of your searches are for strings starting with "AB", you might define a prefix index to store only the first two characters.
Create an index using the INDEX keyword followed by the column and the prefix length in parentheses:
SQL
CREATE INDEX idx_prefix ON table_name(column_name(2));
Test the performance impact of different prefix lengths through actual query execution plans.

#### Example Scenario

Suppose you have a VARCHAR(255) column named description and frequently run queries like:
SQL
SELECT * FROM items WHERE description LIKE 'ABC%';
By creating a prefix index on the first 3 characters, the storage requirement is reduced while still optimizing the query.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: Prefix indexes in MySQL are useful for optimizing VARCHAR columns where queries often search based on initial substrings. They reduce storage and improve performance but require careful selection of prefix length.

#### Common Mistakes
Using an unnecessarily long prefix, leading to reduced space savings.
Ignoring the impact of index fragmentation over time.

#### Follow-Ups
How do you determine the optimal prefix length for your queries?
What are potential trade-offs when using prefix indexes in large datasets?

**Code:**

```sql
CREATE INDEX idx_prefix ON items(description(3));
```

