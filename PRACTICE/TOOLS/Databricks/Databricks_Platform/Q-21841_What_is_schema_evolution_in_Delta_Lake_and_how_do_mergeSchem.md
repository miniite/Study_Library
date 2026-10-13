---
type: "practice"
difficulty: "Intermediate"
---

# What is schema evolution in Delta Lake and how do mergeSchema and overwriteSchema options behave?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Delta Lake in Databricks](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Lake_in_Databricks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Schema evolution in Delta Lake allows for flexible changes to data schemas without requiring a full reprocessing of all historical data. This feature is crucial for maintaining consistency and reducing downtime during schema modifications. The mergeSchema and overwriteSchema options are key mechanisms to achieve this flexibility, each with its own behavior and implications.

#### Triaging the Question
The question asks you to explain schema evolution in Delta Lake, focusing on how the mergeSchema and overwriteSchema operations function. The interviewer is probing your understanding of the internal workings of Delta Lake's schema management capabilities and how these operations impact data integrity and performance.

Schema Evolution in Delta Lake

Delta Lake is built on top of Apache Parquet to provide ACID transactions, versioning, and scalability for big data applications. One of its core features is schema evolution, which allows you to add columns, rename columns, or drop columns from a schema without having to reprocess the entire dataset.

Understanding mergeSchema and overwriteSchema
mergeSchema
Description: The mergeSchema option in Delta Lake merges a new schema with an existing one. If the new schema has additional fields not present in the current schema, they are added to the table. This operation is non-destructive; it does not remove any columns from the original schema.
Behavior:
Adding Columns: You can add new columns to the table by specifying a schema with more fields than currently exist.
No Column Removal: Existing columns will remain unchanged unless explicitly removed using a DROP COLUMN command.

#### Example
SQL
ALTER TABLE my_table MERGE SCHEMA INTO (
  id: INT,
  name: STRING,
  age: INT, -- New column added
  country: STRING
);
overwriteSchema
Description: The overwriteSchema option in Delta Lake completely replaces the existing schema with a new one. This operation is destructive and removes all columns not present in the new schema.
Behavior:
Full Replacement: All existing columns will be dropped, except those explicitly specified in the new schema.

#### Example
SQL
ALTER TABLE my_table OVERWRITE SCHEMA AS (
  id: INT,
  name: STRING
);
Step-by-Step Reasoning from First Principles
Schema Definition: Begin by defining a basic Delta Lake table with an initial schema.
SQL
CREATE TABLE IF NOT EXISTS my_table (id INT, name STRING);
Adding a Column via mergeSchema:
Suppose we want to add a new column age without removing the existing columns.
SQL
ALTER TABLE my_table MERGE SCHEMA INTO (
  id: INT,
  name: STRING,
  age: INT -- New column added
);
Overwriting the Schema via overwriteSchema:
Now, if we need to completely redefine the schema by removing age and adding a new column country.
SQL
ALTER TABLE my_table OVERWRITE SCHEMA AS (
  id: INT,
  name: STRING,
  country: STRING -- Overwrite existing schema with new one
);

#### Closing Summary
and Common Mistakes
Key Takeaways:
Use mergeSchema when you want to add columns without removing existing ones.
Use overwriteSchema for a complete redefinition of the table schema.
Be cautious with overwriteSchema as it can lead to data loss.

#### Common Mistakes
Incorrect Schema Merging: Forgetting that mergeSchema only adds new columns, not overwrites existing ones.
Misusing overwriteSchema: Overwriting the schema without considering all dependencies or data integrity.

**Comparison:**

| Operation | Action                            | Outcome |
|-----------|-----------------------------------|---------|
| mergeSchema | Adds columns to the existing schema | Does not remove existing columns |
| overwriteSchema | Replaces the entire schema        | Removes all non-specified columns |

**Code:**

```sql
-- Example SQL to demonstrate mergeSchema and overwriteSchema

-- Initial table creation
CREATE TABLE IF NOT EXISTS my_table (id INT, name STRING);

-- Adding a new column using mergeSchema
ALTER TABLE my_table MERGE SCHEMA INTO (
  id: INT,
  name: STRING,
  age: INT -- New column added
);

-- Overwriting the schema with overwriteSchema
ALTER TABLE my_table OVERWRITE SCHEMA AS (
  id: INT,
  name: STRING,
  country: STRING -- Overwrite existing schema with new one
);
```

**Example input / output:**

```text
-- Input to mergeSchema and overwriteSchema
Initial Table Schema:
CREATE TABLE IF NOT EXISTS my_table (id INT, name STRING);

SQL for adding age column using mergeSchema:
ALTER TABLE my_table MERGE SCHEMA INTO (
  id: INT,
  name: STRING,
  age: INT -- New column added
);

SQL for overwriting the schema with overwriteSchema:
ALTER TABLE my_table OVERWRITE SCHEMA AS (
  id: INT,
  name: STRING,
  country: STRING -- Overwrite existing schema with new one
);
```

**Data:**

```json
{
  "table": {
    "columns": [
      {"name": "id", "type": "INT"},
      {"name": "name", "type": "STRING"}
    ]
  }
}
```

**Flow:**

1. Initial Schema
2. MERGE SCHEMA
3. New Column Added, ExistingColumns Unchanged
4. OVERWRITE SCHEMA
5. All Non-Specified ColumnsRemoved

