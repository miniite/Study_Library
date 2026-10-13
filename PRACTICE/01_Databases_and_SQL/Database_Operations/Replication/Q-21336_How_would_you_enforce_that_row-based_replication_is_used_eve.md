---
type: "practice"
difficulty: "Deep"
---

# How would you enforce that row-based replication is used everywhere PK-less tables exist, and what's your migration plan to eliminate PK-less tables org-wide?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Changes and Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/Schema_Changes_and_Failover.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce row-based replication for all PK-less (without primary key) tables in a database, we need to ensure that the master-slave replication is configured correctly and all modifications are applied at the row level. This involves understanding the schema, modifying the replication settings, and possibly refactoring the application code or database schema. Additionally, our migration plan should include identifying PK-less tables, converting them where necessary, and maintaining a consistent state across all nodes.

The core of this problem is ensuring data consistency and integrity across the master-slave setup by using row-based replication for those tables that lack primary keys. We need to understand how the database handles transactions and what impact changes in the schema have on replication.

**Comparison:**

| Table Name | Has PK |
|------------|--------|
| orders     | true   |
| products   | true   |
| comments   | false  |

**Code:**

```sql
-- Example SQL to identify PK-less tables
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'your_database' AND table_type = 'BASE TABLE'
AND NOT EXISTS (SELECT 1 FROM information_schema.key_column_usage WHERE referenced_table_schema = 'your_database' AND referenced_table_name = table_name);
```

**Example input / output:**

```

Given a list of tables, we can identify which ones require row-based replication:

SQL
Copy
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'your_database' AND table_type = 'BASE TABLE'
AND NOT EXISTS (SELECT 1 FROM information_schema.key_column_usage WHERE referenced_table_schema = 'your_database' AND referenced_table_name = table_name);

The output might look like:

PLAINTEXT
Copy
+------------+
| table_name |
+------------+
| comments   |
+------------+
```

**Data:**

```json
{
  "tables": [
    { "name": "orders", "has_pk": true },
    { "name": "products", "has_pk": true },
    { "name": "comments", "has_pk": false }
  ]
}
```

**Flow:**

1. Identify PK-less tables
2. Migrate to row-basedreplication
3. Maintain consistent stateacross nodes
4. Monitor and optimize

