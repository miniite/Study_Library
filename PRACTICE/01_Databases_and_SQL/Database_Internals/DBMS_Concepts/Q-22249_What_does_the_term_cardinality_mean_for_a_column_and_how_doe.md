---
type: "practice"
difficulty: "Intermediate"
---

# What does the term "cardinality" mean for a column and how does it affect index selectivity?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Index Access Paths](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Index_Access_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Cardinality refers to the number of unique values in a database column, which is crucial for understanding how efficiently an index can be used during query execution. For a backend developer working with databases, knowing cardinality helps in designing efficient schemas and indexes that improve query performance.

The term "cardinality" comes into play when discussing index selectivity. Index selectivity measures the effectiveness of an index by estimating the number of rows returned for a given query condition. High cardinality columns are those with many distinct values, while low cardinality columns have few distinct values. The selectivity of an index is inversely related to its cardinality: indexes on high-cardinality columns tend to be more selective and thus more effective.

To understand this better, let's break it down step-by-step:

#### Triaging the Question
What is Cardinality? - This refers to the number of unique values in a column.
How does it affect index selectivity? - The cardinality affects how effectively an index can be used during query execution. High-cardinality columns are more likely to have distinct values, making indexes on these columns more selective and thus useful.
Key Assumptions
We assume that the database is a relational database management system (RDBMS) like PostgreSQL or MySQL.
The database schema is well-defined with various tables and indices.
We consider basic SQL queries and their performance implications.

#### First Principles Solution
Cardinality in Columns

Cardinality can be measured as either low, medium, or high:

High Cardinality: A column where the number of distinct values is close to the number of rows. For example, a user_id column in a users table.
Medium Cardinality: The number of distinct values is significantly less than the total number of rows. An example might be a country_code column with only 200 possible codes for 1 million users.
Low Cardinality: A column where there are few distinct values compared to the total number of rows. For instance, a status column in an orders table which can take on one of three states: "pending", "shipped", or "delivered".
Index Selectivity

Index selectivity is calculated as: [ \text{Selectivity} = \frac{\text{Number of distinct values}}{\text{Total number of rows}} ]

For high-cardinality columns, the selectivity will be close to 1, meaning that the index can quickly filter out a large portion of the data. Conversely, low-cardinality columns have much lower selectivity, making indexes less effective.

Impact on Index Use

High Cardinality:

An index on a high-cardinality column is likely to reduce the number of rows examined significantly because it will quickly narrow down the search space.

#### Example
A query like SELECT * FROM users WHERE user_id = '12345' can be very efficient if there's an index on user_id.

Low Cardinality:

An index on a low-cardinality column may not provide much benefit. The database might have to examine most of the rows anyway.

#### Example
A query like SELECT * FROM orders WHERE status = 'pending' might be slow even with an index on status if there are only three possible statuses and many orders.
Practical Considerations
Indexing Strategy: When designing a schema, you should consider columns with high cardinality for indexing. Low-cardinality columns may not need indexes or could benefit from other optimizations.
Query Optimization: Understanding the cardinality of your data helps in writing efficient queries that leverage appropriate indices.
Data Distribution: Ensure that data is well-distributed to avoid skewing and improve query performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, cardinality directly impacts index selectivity. High-cardinality columns are more effective for indexing because they can quickly filter out large portions of the data, whereas low-cardinality columns may not provide much benefit due to their limited distinct values. Understanding these principles is crucial for backend developers to optimize database performance and write efficient queries.

#### Common mistakes
include over-indexing low-cardinality columns or underestimating the value of high-cardinality indices. Always consider the distribution and cardinality when making decisions about indexing.

When asked follow-up questions, you might be probed on specific use cases or asked how to handle skewed data distributions.

