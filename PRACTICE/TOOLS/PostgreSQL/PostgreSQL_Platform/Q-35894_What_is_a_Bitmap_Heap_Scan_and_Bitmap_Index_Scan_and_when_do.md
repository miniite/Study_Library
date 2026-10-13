---
type: "practice"
difficulty: "Expert"
---

# What is a Bitmap Heap Scan and Bitmap Index Scan, and when does the planner prefer it over a plain Index Scan?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Indexing Multi-join Queries](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Indexing_Multi-join_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, Bitmap Heap Scan and Bitmap Index Scan are query execution plans that leverage bitmaps to optimize large result sets, typically in scenarios involving complex queries with multiple predicates or when dealing with high cardinality indexes. These plans are preferred by the query planner over plain index scans under certain conditions.

#### Triage

The question revolves around understanding two specific query execution strategies (Bitmap Heap Scan and Bitmap Index Scan) used by PostgreSQL's query optimizer, and identifying scenarios where these strategies are favored over traditional index scans. The core of this concept lies in understanding how bitmaps can be utilized to optimize disk I/O operations.

#### First-Principles Solution
Bitmap Heap Scan:

A Bitmap Heap Scan is an execution plan used when the planner determines that a sequential scan on the heap (table data) will be more efficient than accessing the table through indexes. This occurs in queries with multiple predicates or complex conditions, where creating and using bitmaps can significantly reduce the number of disk accesses.

Bitmap Index Scan:

A Bitmap Index Scan is used when the planner decides that scanning an index with a bitmap would be faster than performing a full table scan. It involves generating bitmaps for each indexed column in the query predicates and then intersecting these bitmaps to identify matching rows on the heap.

When Does the Planner Prefer These Plans?

The PostgreSQL query planner chooses between Bitmap Heap Scan and Bitmap Index Scan, or uses them over plain index scans, based on several factors:

Multiple Predicates: If a query involves multiple predicates, especially with high cardinality columns, bitmaps can significantly reduce the number of disk accesses by filtering out irrelevant rows in advance.
High Cardinality: For columns where each value is unique or nearly unique, bitmaps are highly effective because they can efficiently manage large sets of row IDs.
Large Result Sets: When dealing with result sets that contain a significant fraction of the table's rows, Bitmap Heap Scan can be more efficient than plain index scans.
Detailed Reasoning

#### Cons
ider a query like:

SQL
SELECT * FROM employees WHERE department = 'Engineering' AND salary > 100000;

Here, the planner might use a Bitmap Index Scan on the department and salary indexes. It would first generate bitmaps for each predicate (i.e., rows where department is 'Engineering' and rows with salary > 100000) and then intersect these bitmaps to get the final set of row IDs.

These row IDs are then used in a Bitmap Heap Scan on the heap, which only reads the actual data from those identified by the bitmap. This process can be more efficient than a plain index scan because it reduces the number of disk accesses required.

#### Closing Summary

In summary, PostgreSQL uses Bitmap Heap Scan and Bitmap Index Scan to optimize queries with multiple conditions or high cardinality indexes. The planner prefers these strategies over plain index scans when dealing with large result sets and complex predicates, as they can significantly reduce the I/O operations needed to retrieve the data.

#### Common Mistakes
and Follow-Ups
Misunderstanding Predicate Count: Sometimes developers underestimate the complexity of queries involving multiple conditions.
Ignoring Index Cardinality: Not considering the cardinality of indexed columns can lead to suboptimal execution plans.
Performance Overhead: While bitmaps reduce I/O, they introduce overhead in generating and managing the bitmaps themselves.

When discussing these topics with an interviewer, it's important to consider real-world scenarios where such optimizations are crucial, like E-commerce platforms handling millions of products or financial systems needing precise transaction histories.

**Comparison:**

```
Query Plan
Bitmap Index Scan onemployees
Generating bitmaps fordepartment and salarypredicates
Intersecting bitmaps to getrow IDs
Bitmap Heap Scan onemployees (using generatedrow IDs)
```

**Code:**

```sql
-- Example query using Bitmap Index Scan
EXPLAIN ANALYZE SELECT * FROM employees WHERE department = 'Engineering' AND salary > 100000;
```

**Example input / output:**

```text
Explanation:
Bitmap Index Scan on employees (cost=0.00..28.46 rows=7 width=29)

Rows Executed: Bitmap Heap Scan on employees (cost=34.16..258.24 rows=2 width=29)
  Recheck Cond: ((department = 'Engineering'::text) AND (salary > 100000))
  ->  Index Only Scan using idx_department_salary on employees (cost=0.00..28.46 rows=7 width=23)
        Index Cond: ((department = 'Engineering'::text) AND (salary > 100000))
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "Alice", "department": "Engineering", "salary": 95000},
    {"id": 2, "name": "Bob", "department": "Marketing", "salary": 120000},
    {"id": 3, "name": "Charlie", "department": "Engineering", "salary": 105000}
  ]
}
```

**Flow:**

1. Query
2. Engineer
3. Choose Bitmap Index/HeapScan
4. Generate Bitmaps
5. Intersect Bitmaps
6. Read Rows

