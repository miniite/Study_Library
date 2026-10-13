---
type: "practice"
difficulty: "Deep"
---

# How would you combine a pgvector similarity search with a metadata WHERE filter (tenant_id, doc_type) so retrieval stays tenant-scoped, and why can a naive filter defeat the vector index?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Vector Similarity in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to ensure that our PostgreSQL database query combines both vector similarity searches and metadata filters (specifically tenant_id and doc_type) while maintaining tenant-scoped retrieval. A naive approach where these conditions are not properly integrated can lead to suboptimal performance or even degradation of the benefits provided by the vector index.

To achieve this, we will leverage PostgreSQL’s powerful query capabilities along with its extension for vector similarity searches (pgvector). We need to ensure that our WHERE clause respects the tenant scope and integrates seamlessly with the vector search. Let's break down the problem step-by-step from first principles.

#### Triage

The key here is ensuring that both the vector index and metadata filters work together effectively without bypassing the vector index’s benefits. The query needs to be structured such that:

Tenant-specific conditions are applied first.
Vector similarity searches are conducted within those tenant boundaries.

A naive filter could defeat the vector index by forcing a sequential scan, which would ignore the vector index and lead to slower performance.

#### First-Principles Solution

We will write a SQL query that first filters on tenant_id and doc_type, then performs the vector similarity search. Let's assume we have a table called documents with columns id, content_vector, tenant_id, and doc_type.

Tenant-specific filtering: Apply tenant-id and doc-type conditions.
Vector similarity search: Use the @> (contains) operator to find documents whose vectors are similar enough.

Here’s how you might write this query in SQL:

SQL
SELECT id, content_vector, tenant_id, doc_type
FROM documents
WHERE tenant_id = :tenant_id
  AND doc_type IN (:doc_types)
  AND content_vector @> vector(:query_vector);
Explanation
tenant_id: This ensures that only documents belonging to the specified tenant are considered.
doc_type IN (... ): Filters based on document type. You can replace this with a list of valid types or use a parameterized approach if there are multiple types.
content_vector @> vector(:query_vector): Performs a similarity search using the vector index.
Full Query Example
SQL
SELECT id, content_vector, tenant_id, doc_type
FROM documents
WHERE tenant_id = 'tenant123'  -- Replace with actual parameter
  AND doc_type IN ('typeA', 'typeB')  -- Replace with actual types
  AND content_vector @> vector('0.1, -0.5, ...');  -- Vector query example

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the vector index is only used for documents within the specified tenant scope and type. A common mistake would be applying doc_type or other filters after the vector search, which could bypass the benefits of the index.

Why a Naive Filter Can Defeat the Vector Index If you apply filters like tenant_id and doc_type after the vector similarity check (e.g., using AND with the vector condition), PostgreSQL might not use the index effectively. This is because the vector index cannot be used when the filter conditions come later in the query plan, leading to a full table scan or sequential search.

#### Follow-Ups

How would you handle partial matches for doc_type?
What are some optimizations we can apply if tenant_id and doc_type have many possible values?
How would this change if you had multiple vector columns?

**Comparison:**

| id | content_vector | tenant_id | doc_type |
|----|---------------|-----------|----------|
| 1  | [0.1, -0.5]    | tenant123 | typeA    |
| 2  | [0.2, -0.6]    | tenant456 | typeB    |

**Code:**

```sql
SELECT id, content_vector, tenant_id, doc_type
FROM documents
WHERE tenant_id = 'tenant123'
  AND doc_type IN ('typeA', 'typeB')
  AND content_vector @> vector('0.1, -0.5, ...');
```

**Example input / output:**

```text
Input:
- tenant_id = 'tenant123'
- doc_types = ('typeA', 'typeB')
- query_vector = [0.1, -0.5]

Output:
- [
  { "id": 1, "content_vector": [0.1, -0.5], "tenant_id": "tenant123", "doc_type": "typeA" }
]
```

**Data:**

```json
{
  "documents": [
    {
      "id": 1,
      "content_vector": [0.1, -0.5],
      "tenant_id": "tenant123",
      "doc_type": "typeA"
    },
    {
      "id": 2,
      "content_vector": [0.2, -0.6],
      "tenant_id": "tenant456",
      "doc_type": "typeB"
    }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Input: Tenant ID, Doc Types,Query Vector
5. Tenant ID Filter?
6. Doc Type Filter?
7. Vector Similarity Search
8. Output: Matching Documents
9. Sequential Scan/Full TableScan

