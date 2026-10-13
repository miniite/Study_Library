---
type: "practice"
difficulty: "Deep"
---

# When is a merge join preferable, and what precondition on the inputs does it need?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Merge joins are a powerful technique in database query optimization but are not always the best choice. This answer will explore when a merge join is preferable and the preconditions required to use this type of join effectively.

#### Triaging the Question

The question asks about conditions under which a merge join should be used, specifically focusing on its applicability. A merge join is an inner join that works by comparing keys from two sorted inputs. It requires the tables to be sorted or have suitable indexes. The key idea here is understanding the performance characteristics and constraints of a merge join.

#### First-Principles Solution
When Is a Merge Join Preferable?

A merge join is preferable when:

Both Inputs Are Already Sorted: This is the primary requirement for a merge join to function correctly.
Cardinality and Data Distribution: The join operation involves comparing keys from both tables, which becomes more efficient when the tables are well-distributed or have low cardinality.
Preconditions on the Inputs

To ensure that a merge join can be effectively used:

Sorted Data: Both input relations must be sorted based on the join key(s). This is often facilitated by index scans.
Index Existence and Alignment: The existence of indexes on the join keys in both tables is crucial, as these indexes enable efficient sorting.
Join Key Cardinality: High cardinality can lead to a large number of comparisons, making other join methods more preferable.
Detailed Analysis

Sorted Data Requirement:

If data is not sorted, a merge join would need to sort the input tables first, which could be costly in terms of time and resources.
For example, if you have two relations A and B, both must be sorted on their respective keys before the merge join can start.

Index Existence:

Indexes provide a way to efficiently sort data without sorting the entire table. This is critical for performance.
For instance, an index on the join key in one of the tables (or both) can significantly reduce the overhead of sorting.

Join Key Cardinality:

High cardinality means many distinct values, leading to a large number of comparisons and potentially making other join methods more efficient.
Low cardinality ensures that fewer comparisons are needed, making a merge join attractive. For example, if A has 10 unique keys and B has 20 unique keys, the total number of comparisons would be relatively low.

#### Closing Summary

In summary, a merge join is preferable when both input tables are sorted or can be efficiently sorted using existing indexes, and the data distribution allows for efficient key comparisons. The preconditions include having sorted data or suitable indexes on the join keys and ensuring that the join operation does not become overly costly due to high cardinality.

#### Common Mistakes
and Follow-Ups
Assuming Sorted Data: Forgetting to check if the input tables are already sorted can lead to unnecessary overhead.
Overlooking Indexes: Not considering the existence of indexes can result in suboptimal query performance.
Ignoring Cardinality: High cardinality can make a merge join less effective, prompting consideration of alternative join methods.

#### Example Scenario

#### Cons
ider two relations:

Orders with index on customer_id
Customers with index on id

If you perform a join on customer_id, both tables are already sorted and an efficient merge join can be used. However, if the Orders table has high cardinality on customer_id, a hash join might be more suitable despite the indexes.

**Comparison:**

| Table Name: Orders |
| id, customer_id |
| Table Name: Customers |
| id, name |

**Code:**

```sql
-- Example SQL query using merge join
SELECT * FROM Orders o JOIN Customers c ON o.customer_id = c.id;
```

**Example input / output:**

```text
# Input data for merge join
Orders Table (sorted on customer_id):
| id | customer_id |
|----|-------------|
| 1  | 101         |
| 2  | 102         |

Customers Table (sorted on id):
| id | name        |
|----|-------------|
| 101| Alice       |
| 102| Bob         |

# Expected Output
| Orders.id | Orders.customer_id | Customers.name |
|-----------|--------------------|----------------|
| 1         | 101                | Alice          |
| 2         | 102                | Bob            |
```

**Data:**

```yaml
# Sample data schema for Orders and Customers tables
orders:
  - id: 1
    customer_id: 101
  - id: 2
    customer_id: 102

customers:
  - id: 101
    name: Alice
  - id: 102
    name: Bob
```

**Flow:**

1. Yes
2. No
3. Orders Sorted on Customer ID
4. Is Index Available?
5. Perform Merge Join
6. Sort and then Perform MergeJoin
7. Output Results
8. Sort Data First, then PerformMerge Join

