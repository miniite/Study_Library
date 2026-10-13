---
type: "practice"
difficulty: "Expert"
---

# In Apache Pinot, how do you choose among sorted, inverted, star-tree, and range indexes for a given query pattern in a real-time analytics table?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ JSON and Text Search Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/JSON_and_Text_Search_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To choose the right index type in Apache Pinot for a given query pattern, we need to consider several factors including query patterns, data characteristics, and performance requirements. The key is understanding how each index works and aligning it with the specific needs of your queries.

#### Triaging the Question

The question revolves around selecting an appropriate index type (sorted, inverted, star-tree, or range) in Apache Pinot for optimizing real-time analytics queries. Apache Pinot supports multiple index types to enable efficient query execution on large-scale data. Each index type has its strengths and is suited to different query patterns.

#### Assumptions
Query Patterns: Understanding the nature of your queries (e.g., select, filter, aggregate).
Data Characteristics: Knowing the structure, distribution, and volume of the data.
Performance Requirements: Balancing between read performance and write performance.
Index Maintenance Cost: Considering the overhead of maintaining indexes.

#### First-Principles Solution

Sorted Index:

Use Case: Ideal for range queries on sorted columns or when the data is already in a lexicographical order.
How It Works: The index allows for efficient range scans, which are common in real-time analytics where you might be filtering based on time or ID ranges.

Inverted Index:

Use Case: Suitable for full-text search and high-cardinality categorical data (e.g., tags, categories).
How It Works: This index maps documents to terms, making it efficient for queries that involve multiple terms or filters on categorical data.

Star-Tree Index:

Use Case: Best for multi-dimensional data where you need fast access to slices of the data (e.g., geographic queries).
How It Works: The index provides a hierarchical structure, allowing efficient traversal and slicing of multidimensional data spaces.

Range Index:

Use Case: Useful when dealing with range-based filters or aggregations.
How It Works: This index supports fast range scans, making it ideal for queries that involve filtering based on ranges of values (e.g., date ranges).
Reasoning Step-by-Step
Identify Query Patterns:
Determine if your primary queries are range-based, full-text search, multidimensional slicing, or categorical filters.
Analyze Data Characteristics:
Check the distribution and cardinality of your data fields.
Evaluate Performance Requirements:
Assess whether read performance is more critical than write performance.

#### Cons
ider Index Maintenance Costs:
Ensure that the overhead of maintaining an index does not outweigh its benefits.

#### Example Scenario

Suppose you have a real-time analytics table with data about user activities, where users can filter by date range and tag categories. You might choose:

A Sorted Index for time-based queries.
An Inverted Index for filtering based on tags (high-cardinality categorical data).
A Range Index for date ranges.
No need for a Star-Tree Index unless you expect frequent multidimensional slicing.

#### Closing Summary
and Common Mistakes

Choosing the right index involves a trade-off between query performance, data distribution, and maintenance costs. The key is to align your choice with the specific needs of your queries. A common mistake is to over-index, which can lead to increased storage and maintenance overhead without significant performance gains.

