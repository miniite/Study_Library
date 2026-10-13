---
type: "practice"
difficulty: "Deep"
---

# Explain how you would migrate a wide table's rarely-used columns into a separate 1:1 table (vertical split) without breaking existing queries during rollout.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating a wide table with rarely-used columns to a separate 1:1 table, we aim to minimize downtime and ensure that existing queries continue to function seamlessly. This involves carefully planning the migration strategy, using database-specific features like triggers or transactions, and ensuring all changes are thoroughly tested before rollout.

The core idea is to create a new detail table with the rarely-used columns and then link it via a foreign key to the original main table. During the migration, we update queries that reference the old wide table's columns to join with the detail table instead. We also need to manage any existing relationships or constraints in the application code.

Let’s walk through the steps from first principles:

Identify the Columns for Splitting:

Analyze which columns are rarely used.
Determine if these columns can be moved without affecting the primary business logic of the application.

Create the New detail Table:

Design a new table that includes only the rarely-used columns from the original wide table.
Ensure it has a foreign key referencing the main table (let's call it main_id) to maintain 1:1 relationship.

Update Existing Queries and Code:

Update all SELECT, INSERT, UPDATE, DELETE queries that reference these rarely-used columns to join with the new detail table.
Modify stored procedures, functions, or application code accordingly.

Test Thoroughly Before Rollout:

Perform unit tests on individual components.
Execute end-to-end integration tests to ensure all processes work as expected.
Conduct load testing to check performance under normal and peak conditions.

Rollout Strategy:

Use a phased approach if possible, starting with non-critical queries first.
Monitor the system closely during the rollout phase for any issues that may arise.

Post-Migration Cleanup:

Remove old columns from the main table once migration is confirmed successful in all environments.
Update documentation to reflect changes.

**Comparison:**

```
has
details
MAIN_TABLE
DETAIL_TABLE
detail
```

**Code:**

```sql
-- Step 2: Create the new detail table
CREATE TABLE IF NOT EXISTS `main_detail` (
    `detail_id` INT PRIMARY KEY,
    `main_id` INT NOT NULL,
    FOREIGN KEY (`main_id`) REFERENCES `main_table`(`id`)
);
```

**Example input / output:**

```text
-- Example of updating a query to include the detail table

-- Original Query
SELECT id, common_column_1 FROM main_table WHERE condition;

-- Updated Query
SELECT m.id, m.common_column_1, d.rarely_used_column_A, d.rarely_used_column_B
FROM main_table m
JOIN main_detail d ON m.id = d.main_id
WHERE condition;
```

**Data:**

```json
{
  "main_table": {
    "columns": [
      { "name": "id", "type": "INT" },
      { "name": "common_column_1", "type": "VARCHAR(255)" },
      // ... other commonly used columns ...
      { "name": "rarely_used_column_A", "type": "TEXT" },
      { "name": "rarely_used_column_B", "type": "DATE" }
    ]
  },
  "main_detail": {
    "columns": [
      { "name": "detail_id", "type": "INT" },
      { "name": "main_id", "type": "INT" },
      { "name": "rarely_used_column_A", "type": "TEXT" },
      { "name": "rarely_used_column_B", "type": "DATE" }
    ]
  }
}
```

**Flow:**

1. Identify Columns
2. Create Detail Table
3. Update Queries
4. Test Thoroughly
5. Rollout Strategy
6. Cleanup Old Columns
7. DONE

