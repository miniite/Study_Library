---
type: "practice"
difficulty: "Deep"
---

# What is the difference between cosine distance and L2 distance operators in pgvector, and which fits normalized embeddings?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Vector Similarity in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL's pgvector extension, both cosine distance and L2 distance are essential metrics used to measure similarity or dissimilarity between vectors. The key difference lies in their mathematical formulation and use cases, particularly when dealing with normalized embeddings.

TL;DR

Cosine distance measures the angular difference between two vectors, making it ideal for normalized embeddings where direction is more important than magnitude. L2 distance, on the other hand, measures the Euclidean distance directly, suitable for vectors of varying magnitudes.

#### Triaging the Question

The question asks about the differences between cosine and L2 distances in pgvector. It's probing your understanding of vector similarity metrics and their applicability to normalized embeddings. We assume that both operations are part of a database query where vector similarity is needed, and we need to choose which metric fits better for normalized data.

#### First-Principles Solution

Cosine Distance:

Definition: Cosine distance between two vectors (\mathbf{a}) and (\mathbf{b}) is calculated as ( 1 - \cos(\theta) = 1 - \frac{\mathbf{a} \cdot \mathbf{b}}{|\mathbf{a}| |\mathbf{b}|} ).
Application: This measure focuses on the angle between vectors, making it highly suitable for normalized embeddings where each vector's magnitude is unity. The cosine distance essentially measures how aligned two vectors are.

L2 Distance:

Definition: L2 distance (Euclidean distance) between two vectors (\mathbf{a}) and (\mathbf{b}) is calculated as ( |\mathbf{a} - \mathbf{b}|2 = \sqrt{\sum{i=1}^{n} (a_i - b_i)^2} ).
Application: This measure considers the overall magnitude of the vectors, making it useful for data where both direction and magnitude are significant.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, when dealing with normalized embeddings in pgvector, cosine distance is more appropriate because it focuses on the angle between vectors, which aligns well with the properties of normalized data. However, if you need to consider vector magnitudes or work with non-normalized data, L2 distance would be a better choice.

A common mistake might be assuming that both metrics are interchangeable without considering the implications of their mathematical differences. Always ensure the metric chosen reflects the underlying characteristics and requirements of your data.

Common Follow-Ups
How do you implement these distance metrics in pgvector?
What are the performance considerations when using these metrics on large datasets?
Are there any specific use cases where one might be preferred over the other?

**Comparison:**

```
Vector A	Vector B	Cosine Distance	L2 Distance
[0.5, 0.7]	[0.6, 0.8]	0.143	0.141
```

**Code:**

```sql
-- Example of calculating cosine distance
SELECT pgvector.cosine_distance(v1, v2) FROM (VALUES ('[0.5, 0.7]', '[0.6, 0.8]')) AS t(v1, v2);

-- Example of calculating L2 distance
SELECT pgvector.l2_distance(v1, v2) FROM (VALUES ('[0.5, 0.7]', '[0.6, 0.8]')) AS t(v1, v2);
```

**Example input / output:**

```text
-- Example input and output for cosine distance
Input:
SELECT pgvector.cosine_distance('[0.5, 0.7]', '[0.6, 0.8]');

Output:
Result: 0.143

-- Example input and output for L2 distance
Input:
SELECT pgvector.l2_distance('[0.5, 0.7]', '[0.6, 0.8]');

Output:
Result: 0.141
```

**Data:**

```json
{
  "vectors": [
    {"id": 1, "vector": [0.5, 0.7]},
    {"id": 2, "vector": [0.6, 0.8]}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Start
4. Cosine Distance Measure
5. Is vector magnitudesignificant?
6. Cosine Distance
7. L2 Distance
8. Use for normalized data
9. Consider both direction andmagnitude
10. Example: Cosine distance for[0.5, 0.7] vs [0.6, 0.8]
11. Example: L2 distance for [0.5,0.7] vs [0.6, 0.8]
12. Result: 0.143
13. Result: 0.141

