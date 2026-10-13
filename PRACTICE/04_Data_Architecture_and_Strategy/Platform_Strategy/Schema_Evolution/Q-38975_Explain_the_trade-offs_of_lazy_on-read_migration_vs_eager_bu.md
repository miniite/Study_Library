---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-offs of lazy/on-read migration vs eager/bulk backfill for a large denormalized dataset. What operational and correctness risks make you pick one, and can you combine them?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Migrations with Read Your Writes](<../../../../04_Data_Architecture_and_Strategy/Platform_Strategy/Schema_Evolution/Migrations_with_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development and architecture, schema evolution involves changing an existing database schema to adapt to evolving requirements without losing data or causing downtime. When dealing with large denormalized datasets, two common strategies are lazy/on-read migration and eager/bulk backfill. Let’s break down the trade-offs, operational and correctness risks associated with each, and explore how they can be combined.

Lazy/on-read migration means that the schema changes are applied incrementally as data is accessed or modified. On the other hand, eager/bulk backfill involves applying all necessary schema changes in one go during a maintenance window. Each strategy has its own set of advantages and disadvantages, which we will explore step-by-step.

**Comparison:**

|Migration Strategy| Impact on Performance| Data Integrity Considerations|
|---|---|---|
|Lazy/on-read migration| Minimal impact; incremental changes| Ensure data integrity during access, handle concurrent modifications carefully|
|Eager/bulk backfill| Temporary performance degradation during bulk update| No concurrent access issues, but more downtime required|

**Code:**

```
MERMAID
Copy
graph TD;
    A[Lazy/on-read migration] --> B{Does the system support lazy migrations?}
    B -- Yes --> C[Incremental changes, minimal impact on performance]
    B -- No --> D[Eager/bulk backfill is required]
    C --> E[Ensure data integrity during access]
    D --> F[Bulk update may cause temporary performance degradation]
```

**Example input / output:**

```text
Let's consider a scenario where we need to add a `new_field` to the `large_denormalized_data` dataset and rename the table from `users` to `customers`.

Lazy/on-read migration:
1. When accessing data, if a row doesn’t have `new_field`, it will be lazily loaded.
2. During bulk operations like insertions or updates, only those rows without `new_field` are modified.

Eager/bulk backfill:
1. Perform a one-time update to all existing rows in the dataset to ensure they have the new schema.
2. This could cause temporary performance degradation and might require locking the table.
```

**Data:**

```yaml
- dataset: large_denormalized_data
  schema_changes:
    - type: add_column
      column_name: new_field
      column_type: integer
    - type: rename_table
      old_table_name: users
      new_table_name: customers
```

**Flow:**

1. Yes
2. No
3. Access Data
4. Does the system support lazymigrations?
5. Lazy/on-read migration
6. Eager/bulk backfill
7. Incrementally modify data asneeded, minimal impact onperformance
8. Bulk update all existing rows,temporary performancedegradation

