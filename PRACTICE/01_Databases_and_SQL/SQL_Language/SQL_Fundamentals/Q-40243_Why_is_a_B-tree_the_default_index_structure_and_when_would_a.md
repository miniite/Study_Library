---
type: "practice"
difficulty: "Expert"
---

# Why is a B-tree the default index structure, and when would a hash index be better?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Leftmost Prefix Rule](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why B-trees are the default index structure in databases and when hash indexes might be more appropriate, let's start with the core concepts of each.

B-trees provide efficient lookup, insertion, and deletion operations, making them ideal for database indexing. They maintain sorted order, allowing for fast range queries and supporting multiple keys per node. Hash indexes, on the other hand, use a hash function to map values to specific buckets, enabling very quick lookups but not supporting range queries or sorting.

The key difference lies in their performance characteristics: B-trees are well-suited for scenarios requiring order maintenance and multi-key support, whereas hash indexes excel where fast single-record access is paramount.

Let's delve into the details.

**Comparison:**

| Feature        | B-tree Index                    | Hash Index                      |
|----------------|---------------------------------|--------------------------------|
| Lookup Speed   | Fast, but slower than hash      | Very fast                       |
| Insertion/Deletion | Efficient                     | Less efficient                 |
| Range Queries  | Supported                       | Not supported                   |
| Multiple Keys Per Node | Yes                            | No                             |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Data] --> B[B-tree Index]
    B --> C[Sorted Insertion & Deletion]
    C --> D[Range Queries Supported]
    B --> E[Multiples Keys per Node]

    F[Data] --> G[Hash Index]
    G --> H[Fast Single-Record Lookup]
    H --> I[No Range Query Support]
```

**Example input / output:**

```

Consider a database of customer records. You want to implement an index for fast lookups by customer ID, while also supporting range queries on customer names.

B-tree Index: Ideal as it supports both customer_id lookups and name range queries.

SQL
Copy
CREATE INDEX idx_customer_id ON customers(customer_id);

Hash Index: Suitable if you only need to quickly find a single record by customer ID but do not require any range queries.

SQL
Copy
CREATE INDEX idx_customer_id_hash ON customers USING hash (customer_id);
```

**Data:**

```json
{
  "BTree": {
    "name": "B-tree",
    "operations": ["lookup", "insertion", "deletion"],
    "features": [
      {"key": "sorted", "value": true},
      {"key": "range_queries", "value": true},
      {"key": "multiple_keys_per_node", "value": true}
    ]
  },
  "HashIndex": {
    "name": "Hash Index",
    "operations": ["lookup"],
    "features": [
      {"key": "fast_single_record_lookup", "value": true},
      {"key": "range_queries", "value": false}
    ]
  }
}
```

**Flow:**

1. Database Operation
2. B-tree Index
3. Insert Data
4. Sorted Insertion
5. Return Success
6. A
7. Hash Index
8. Single-Record Lookup
9. Return Data

