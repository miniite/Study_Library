---
type: "practice"
difficulty: "Deep"
---

# How would you use EXPLAIN FORMAT=JSON to spot a hidden filesort or a temporary table that the classic EXPLAIN output hides?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Reading EXPLAIN](<../../../../TOOLS/MySQL/MySQL_Platform/Reading_EXPLAIN.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how EXPLAIN FORMAT=JSON can help us spot a hidden filesort or a temporary table, let's first break down what these terms mean and then dive into using JSON to analyze the query execution plan.

#### Triaging the Question

When you run a query in MySQL and use EXPLAIN, it provides an overview of how the database engine plans to execute the query. The classic EXPLAIN output is great for understanding basic aspects like which indexes are used, but sometimes it doesn't provide enough detail about more complex operations such as filesorts or temporary tables.

The EXPLAIN FORMAT=JSON command offers a richer and more detailed output that can reveal these hidden operations by providing a deep dive into the query execution plan. By examining the JSON output, we can identify specific nodes in the plan that indicate whether a filesort is happening behind the scenes or if a temporary table is being used.

#### First-Principles Solution

Understanding Filesorts: A filesort occurs when MySQL needs to sort data but doesn't have an index available that matches the ORDER BY clause of the query. This can be inefficient because it requires additional disk I/O and memory usage.

Temporary Tables: Temporary tables are used by MySQL in certain operations, such as joins or aggregations, where intermediate results need to be stored temporarily before being processed further.

JSON Output Analysis:

In the JSON output, we can look for specific nodes that indicate these operations.
Nodes labeled with filesort will show us which part of the query is causing a filesort.
Nodes related to temporary tables often have properties like temporary=true.

#### Example
Query and Output:

SQL
EXPLAIN FORMAT=JSON SELECT * FROM customers WHERE customer_id = 10 ORDER BY name;
Detailed JSON Output Interpretation

A typical JSON output might look something like this:

JSON
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "9.532"
    },
    "table": {
      "access_type": "ref",
      "possible_keys": ["customer_id_idx"],
      "key": "customer_id_idx",
      "key_length": "4",
      "used_key_parts": ["customer_id"],
      "rows_examined_per_scan": 1,
      "rows_produced_per_join": 1,
      "filtered": "100.00",
      "extra": "Using index; Using filesort"
    }
  },
  "plans": [
    {
      "table": {
        "access_type": "ref",
        "possible_keys": ["customer_id_idx"],
        "key": "customer_id_idx",
        "key_length": "4",
        "used_key_parts": ["customer_id"],
        "rows_examined_per_scan": 1,
        "rows_produced_per_join": 1,
        "filtered": "100.00",
        "extra": "Using index"
      }
    },
    {
      "table": {
        "access_type": "filesort",
        "filesort_keys": ["'a','b'"],
        "rows_sorted": 2
      }
    }
  ]
}

In this example, the extra field in the table node shows that a filesort is being used. The plans array further breaks down the operations and reveals that there's an additional step involving sorting.

#### Closing Summary

By using EXPLAIN FORMAT=JSON, we can get a deeper understanding of query execution plans, uncovering hidden operations like filesorts or temporary tables. This helps in optimizing queries to reduce unnecessary I/O and memory usage, leading to more efficient database performance.

#### Common Mistakes/Follow-Ups
Misinterpreting the JSON Output: Failing to correctly interpret the extra field or other nodes can lead to incorrect conclusions.
Overlooking Temporary Tables: Sometimes temporary tables are used for operations like joins or aggregations that aren’t immediately obvious from a simple EXPLAIN.
Not Considering Indexes: Always check if there are better indexes that could reduce the need for filesorts.

