---
type: "practice"
difficulty: "Deep"
---

# How do you detect a hidden transitive dependency in a warehouse table that's silently inflating a report and fix it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Transitive Dependencies and 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Transitive_Dependencies_and_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect a hidden transitive dependency in a warehouse table that is silently inflating a report, we need to understand the underlying data relationships and ensure our schema design adheres to normalization principles. A hidden transitive dependency occurs when a dependency between two attributes does not directly appear but can be inferred from other dependencies. Here’s how you would approach this problem:

Firstly, let's clarify what a transitive dependency is: it arises in a many-to-many relationship where there are intermediate tables that need to be joined to establish the direct relationship. If such a dependency is not properly managed, it can lead to data redundancy and inflated reports.

To detect these issues, we would start by analyzing the current schema design, identifying all the foreign key relationships, and then looking for indirect dependencies through multiple joins. Here’s a step-by-step approach:

Schema Analysis: Examine the database schema to identify all tables involved in many-to-many relationships.
Dependency Mapping: Map out the relationships using a dependency diagram or flowchart.
Query Performance Monitoring: Use query performance metrics and logs to pinpoint which queries are running slow due to excessive joins, indicating potential hidden dependencies.
Data Integrity Checks: Run data integrity checks for each table to ensure that all values in related tables conform to expected constraints.

Once we have identified the problematic relationships, we can implement a fix by normalizing or denormalizing the schema appropriately.

**Code:**

```sql
-- Example of detecting transitive dependencies through slow query analysis
SELECT query_text, execution_time FROM performance_schema.events_statements_summary_by_digest WHERE digest_text LIKE '%JOIN%';
```

**Example input / output:**

```text
Input:
- A query that joins the Product and Category tables multiple times due to the transitive dependency.

Output:
- A performance report indicating slow query execution due to excessive joins.
- A recommendation to normalize or denormalize the schema by adding a direct foreign key in the Product table referencing the parent category if needed.
```

**Data:**

```json
{
  "tables": [
    {
      "name": "Product",
      "columns": ["product_id", "name", "category_id"]
    },
    {
      "name": "Category",
      "columns": ["category_id", "parent_category_id", "description"]
    }
  ],
  "foreign_keys": [
    {"table": "Product", "column": "category_id", "ref_table": "Category", "ref_column": "category_id"}
  ]
}
```

