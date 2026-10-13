---
type: "practice"
difficulty: "Intermediate"
---

# How does pgvector let you run a nearest-neighbour similarity search in SQL, and what does the ORDER BY clause look like?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Similarity in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how pgvector allows running a nearest-neighbour similarity search in SQL, we first need to grasp that pgvector is an extension for PostgreSQL that enables vector operations and similarity searches on vectors stored as elements within the database. The core idea revolves around leveraging PostgreSQL's powerful query capabilities to perform these operations directly in SQL.

#### Triaging the Question

The question asks two main parts:

How pgvector facilitates nearest-neighbour similarity search.
What the ORDER BY clause looks like for such a query.

#### Assumptions
We are working with PostgreSQL and have installed and configured the pgvector extension.
Vectors are stored as elements in a table, typically using a data type like vector.
Step-by-Step Solution

Understanding pgvector: The pgvector extension introduces functions for vector operations such as dot product, distance metrics (like L2 norm), and similarity searches.

Nearest-Neighbour Search:

In the context of pgvector, a nearest-neighbour search is finding the vector in a collection that most closely resembles a given query vector.
This can be achieved using functions like vector_l2_distance to calculate distances between vectors and then ordering by these distances.

Using ORDER BY for Similarity Search:

To perform a nearest-neighbour similarity search, we typically use the ORDER BY clause combined with a distance metric function.
For example, using the L2 norm (Euclidean distance), we can find vectors that are closest to a query vector.

#### Example
Code

Here's how you might write such an SQL query:

SQL
SELECT *
FROM vectors_table
ORDER BY vector_l2_distance(query_vector, vector_column) ASC
LIMIT 1;
vectors_table is the table containing your vectors.
query_vector is a constant vector used as the reference for finding similar vectors.
vector_column is the column in vectors_table that stores the vectors.
vector_l2_distance(query_vector, vector_column) calculates the L2 distance between each row's vector and the query vector.
The ORDER BY ... ASC clause sorts the results by increasing distance, effectively bringing the closest matches to the top.
LIMIT 1 limits the output to just one result, which is typically the nearest neighbour.

#### Closing Summary

In summary, pgvector allows running a nearest-neighbour similarity search in SQL through vector operations and functions. The key idea involves using the ORDER BY clause with distance metrics like vector_l2_distance to sort vectors by their similarity to a query vector. This approach leverages PostgreSQL’s powerful querying capabilities directly within the database.

#### Common Mistakes/Follow-Ups
Mistake: Failing to understand that pgvector requires installation and configuration.

#### Follow-Up
Exploring different distance metrics or additional functions like cosine similarity provided by pgvector.

**Code:**

```sql
SELECT *
FROM vectors_table
ORDER BY vector_l2_distance(query_vector, vector_column) ASC
LIMIT 1;
```

**Example input / output:**

```text
-- Sample input
query_vector := (0.5, 0.3, -0.6);

-- Sample output
SELECT * FROM vectors_table ORDER BY vector_l2_distance(query_vector::vector, vector_column) ASC LIMIT 1;
```

