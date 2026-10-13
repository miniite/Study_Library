---
type: "practice"
difficulty: "Deep"
---

# You want to add a case-insensitive unique constraint on email at 100M rows. Compare a functional index on lower(email), a generated column, and CITEXT. Which do you ship and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Functional Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Functional_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to compare three approaches for adding a case-insensitive unique constraint on email in a database with 100 million rows: using a functional index on lower(email), creating a generated column, and utilizing the CITEXT data type. We'll analyze each option based on performance, maintainability, and future scalability.

First, let's define what we're dealing with:

A case-insensitive unique constraint on email means that any two rows cannot have the same email value regardless of its case (e.g., "john@example.com" is considered equal to "JOHN@EXAMPLE.COM").
Functional indexes can be used to create an index on a transformed version of the column.
Generated columns allow us to store and access computed values as if they were regular columns, but at a lower storage cost.
CITEXT is a data type in PostgreSQL that automatically applies case-insensitive comparison rules.

Now, let's dive into each approach:

1. Functional Index on lower(email)

#### Pros

Simplicity: Creating an index on the transformed column (lower(email)) can be straightforward and easy to implement.
Performance: The performance overhead is minimal since the index is based on a simple transformation of the existing column.

#### Cons

Index Size: If you have 100 million rows, each email can be up to 255 characters. Converting email to lower(email) will double or triple the size of the index, potentially leading to significant storage overhead.
Maintenance: Any update to the original email column requires a corresponding update to lower(email), which could lead to performance degradation during updates.
2. Generated Column

#### Pros

Efficiency: A generated column can be created on-the-fly without duplicating data, reducing storage costs.
Maintainability: The transformation logic for the generated column can be encapsulated within a single statement, making it easier to maintain and understand.

#### Cons

Complexity: Implementing a unique constraint using a generated column requires additional complexity in SQL queries.
Performance Overhead: While minimal, there might still be some performance overhead due to the need for the database to compute the lower(email) value on each query.
3. CITEXT Data Type

#### Pros

Simplicity and Efficiency: Using CITEXT can simplify the schema design since it automatically handles case-insensitive comparisons, reducing the need for additional transformations.
Maintainability: The database takes care of all the case sensitivity issues internally, making the application code cleaner.

#### Cons

Performance Impact: While CITEXT is optimized for case-insensitivity, it might still have some performance overhead compared to regular text types, especially on large data sets.
Compatibility: Not all databases support CITEXT. PostgreSQL, however, provides this feature and has excellent performance and reliability.

#### Conclusion

For a database with 100 million rows, the choice between these options depends heavily on the specific use case and constraints:

If storage is a significant concern, and you have the flexibility to handle some additional overhead during updates, using CITEXT might be the best approach. It simplifies your schema and minimizes maintenance.
For a balance of simplicity and efficiency, a generated column could be used with careful consideration of performance implications.
If storage is not a critical factor, but you want to minimize complexity in application code, a functional index on lower(email) can be implemented more straightforwardly.

In conclusion, I would recommend using the CITEXT data type because it offers a clean and efficient solution that aligns well with database best practices. However, the final decision should consider the specific requirements of your project, including storage limitations and maintenance overhead.

