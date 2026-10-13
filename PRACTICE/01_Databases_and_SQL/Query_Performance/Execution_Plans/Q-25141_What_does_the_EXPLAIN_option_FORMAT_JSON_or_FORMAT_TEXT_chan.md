---
type: "practice"
difficulty: "Intermediate"
---

# What does the EXPLAIN option FORMAT JSON or FORMAT TEXT change about the plan output?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reading Execution Plans](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Reading_Execution_Plans.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When using the EXPLAIN option in SQL to get execution plans, specifying either FORMAT JSON or FORMAT TEXT changes how the query optimizer’s decisions are presented and can provide different levels of detail. The primary difference lies in the format and readability of the output.

The FORMAT JSON provides a structured, machine-readable representation that includes detailed information about each step of the execution plan, such as operations like scans, filters, sorts, and join types. This format is useful for automated processing or when you need to analyze the plan using tools designed to parse JSON data.

On the other hand, FORMAT TEXT outputs a human-readable format, which can be easier to understand quickly but lacks some of the structured metadata available in JSON. The textual output still shows the operations and their order, but with less detailed annotations compared to JSON.

**Example input / output:**

```
TEXT Format Output Example:
PLAINTEXT
Copy
Seq Scan on large_table  (cost=0.00..10000.00 rows=1 width=500)
  Filter: (id = 123)
JSON Format Output Example:
JSON
Copy
{
  "Plan": {
    "Node Type": "Seq Scan",
    "Relation Name": "large_table",
    "Alias": "",
    "Startup Cost": 0.00,
    "Total Cost": 10000.00,
    "Plan Rows": 1,
    "Plan Width": 500,
    "Filter": "(id = 123)"
  }
}
```

