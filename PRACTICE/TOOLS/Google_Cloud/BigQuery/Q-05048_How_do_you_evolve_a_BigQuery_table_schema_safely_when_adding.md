---
type: "practice"
difficulty: "Deep"
---

# How do you evolve a BigQuery table schema safely when adding, relaxing, or removing a column, and which changes require a rewrite?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ BigQuery Loading and Table Design](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evolving a BigQuery table schema in a production environment, ensuring data integrity and minimizing downtime are paramount. We need to handle additions, relaxations (changes that do not affect existing data), and removals of columns carefully, as some changes might require rewrites of the entire dataset.

#### Triage

The question is essentially asking how to safely modify the schema of a BigQuery table without causing data loss or corruption. The interviewer wants you to discuss best practices for altering the schema and understand which operations necessitate rewriting the data, such as adding columns with default values, removing non-nullable columns, etc.

Triaging Assumptions
You are using a versioned approach where changes are tracked in a way that allows rollbacks if needed.
The table is not too large to handle incremental schema changes efficiently.
Performance and data consistency are key considerations.

#### First-Principles Solution
Adding a Column

Adding a column is generally safer because it doesn't change existing rows unless the new column has a default value or a specific strategy for handling nulls.

Non-Populated Columns: If you add a non-required (nullable) column, BigQuery will insert NULL values in the old rows.
Populated Columns with Default Values:
Use a query to update all existing rows: ALTER TABLE table_name ADD COLUMN new_column_name TYPE default_value
This can be done in batches if the table is large.
Relaxing a Column

Relaxing columns, such as changing a required column to nullable or altering its type (e.g., from INT64 to STRING), can sometimes require data migrations. Here’s how:

Changing Required to Nullable:
This change is safe and doesn't affect existing rows, but you should still back up the schema before making changes.
Type Changes: Be cautious with type changes as they might lead to loss of data if not handled correctly.
Removing a Column

Removing columns can be trickier because it involves rewriting all old rows to fit into the reduced schema:

Remove Non-Nullable Columns:
BigQuery will raise an error because it cannot truncate non-nullable values.
Remove Nullable Columns:
You need to write a query that updates all existing rows, removing or replacing the column value with NULL.

#### Example
ALTER TABLE table_name DROP COLUMN old_column_name

#### Common Mistakes
Forgetting to Backup: Always back up your schema before making changes.
Ignoring Data Consistency: Ensure that any new columns have appropriate default values and update strategies.
Not Considering Performance: Large tables may require careful planning, especially for non-batch updates.

#### Closing Summary
and Follow-Ups

In summary, evolving BigQuery schemas safely involves understanding the impact of each change and carefully managing data during schema modifications. Adding or relaxing columns can often be done without rewriting all data, but removing a column typically requires reprocessing existing rows to fit the new schema. Always plan your changes with rollback strategies in mind.

Common follow-ups might include:

How do you handle very large tables?
What tools or scripts would you use for batch operations?
How do you ensure data consistency during these changes?

**Comparison:**

```
Column	Type
id	INT64
name	STRING
age	INT64
new_column	STRING (DEFAULT 'default_value')
```

**Code:**

```sql
-- Example of adding a column with default value
ALTER TABLE my_table ADD COLUMN new_column STRING DEFAULT 'default_value';

-- Example of changing a required column to nullable
ALTER TABLE my_table MODIFY COLUMN old_column STRING NULL;
```

**Example input / output:**

```text
Input:
ALTER TABLE my_table ADD COLUMN new_column STRING DEFAULT 'default_value';

Output:
Schema updated successfully.
```

**Data:**

```json
{
  "table_name": "my_large_table",
  "columns": [
    {"name": "id", "type": "INT64"},
    {"name": "name", "type": "STRING"},
    {"name": "age", "type": "INT64"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Is column addition?
9. Add Nullable Column
10. Is column type changerequired?
11. Handle Type Changes, mayneed data migration
12. Is column removal needed?
13. Remove Non-NullableColumns, requiresreprocessing rows
14. Schema update successful
15. Update Schema with NewColumn
16. Perform Data Migration
17. Reprocess Rows to Fit NewSchema

