---
type: "practice"
difficulty: "Deep"
---

# A materialized view with FAST refresh suddenly started doing COMPLETE refreshes in production; what would you check to find why the incremental path was invalidated?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a materialized view (MV) that is supposed to perform fast refreshes starts doing complete refreshes, it indicates that the incremental path has become invalid or unusable. This could be due to various reasons such as schema changes, data modifications, or metadata issues. To diagnose and resolve this issue, we need to systematically investigate the factors affecting the MV's ability to use its incremental refresh mechanism.

**Code:**

```sql
-- Sample code to check MV status and history
SELECT
    view_name,
    last_refresh_type,
    last_refresh_start_time,
    last_refresh_success,
    last_invalidation_reason
FROM
    dba_mviewinvalidations;
```

**Example input / output:**

```text
-- Example output of the query above
VIEW_NAME | LAST_REFRESH_TYPE | LAST_REFRESH_START_TIME | LAST_REFRESH_SUCCESS | LAST_INVALIDATION_REASON
----------|-------------------|-------------------------|----------------------|-----------------------------
mv_sales  | COMPLETE          | 2023-10-15 09:00:00     | NO                   | Schema changed in underlying table
```

**Flow:**

1. Yes
2. No
3. Check MV Status and History
4. Is there a record of aninvalidation?
5. Review Last InvalidationReason
6. Analyze underlying changesaffecting the MV
7. Check MV Refresh Settings
8. Validate MV Dependencies
9. Monitor for Schema Changesor Data Modifications

