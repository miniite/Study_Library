---
type: "practice"
difficulty: "Intermediate"
---

# Your connection pool holds long-lived sessions and a schema change (new column) isn't seen by cached prepared statements, causing errors; explain why and how you'd invalidate or reset the pool safely.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When using a connection pool in PostgreSQL with long-lived sessions, prepared statements might be cached to improve performance. However, if a schema change is made (e.g., adding a new column), these cached prepared statements may not recognize the changes, leading to runtime errors. To address this issue, you need to invalidate or reset the connection pool safely to ensure that all clients use up-to-date information.

#### Triaging the Question

The question revolves around understanding why schema changes are not immediately recognized by cached prepared statements and how to handle this situation effectively. The key aspects being probed include:

Understanding of PostgreSQL's connection pooling mechanism.
Knowledge of how prepared statements work in the context of schema changes.
Ability to diagnose and resolve issues related to stale database objects.

#### First Principles Solution

When a schema change is made, such as adding a new column, the metadata associated with that schema (including prepared statements) is not automatically updated. Therefore, if you have cached prepared statements from previous sessions or long-lived connections, they will still reference the old schema, leading to runtime errors when attempting operations related to the newly added column.

Steps to Diagnose and Resolve

Identify Affected Prepared Statements:

Run a query to identify which prepared statements are stale.
SQL
SELECT stmt FROM pg_prepared_statements;

Invalidate Specific Prepared Statements:

You can invalidate specific prepared statements using the DEALLOCATE command.
SQL
DEALLOCATE PREPARED_STATEMENT_NAME;

Alternatively, you can deallocate all prepared statements:

SQL
DEALLOCTE ALL;

Reset Connection Pool:

If invalidating specific statements does not resolve the issue, consider resetting the entire connection pool.
In PostgreSQL, this can be achieved by restarting the database or using a tool to reset connections, such as pgbouncer.

Monitor and Validate:

After making these changes, monitor the application logs and ensure that no errors persist due to schema mismatches.

#### Closing Summary
and Common Mistakes

To safely handle schema changes in PostgreSQL with long-lived sessions and connection pools:

Invalidate or reset prepared statements to ensure they reflect the latest schema.
Use tools like pgbouncer for managing connection pools effectively.
Monitor application behavior closely after making such changes to catch any residual issues.

#### Common Mistakes/Follow-Ups
Failing to Invalidate Specific Statements:
Invalidation might not always be sufficient if the issue is widespread, leading to potential delays in resolving errors.
Over-Resetting the Connection Pool:
Resetting the entire connection pool can introduce downtime and may affect other applications sharing the same pool.
Lack of Monitoring:
Failing to monitor application behavior after making changes can result in undetected issues that may lead to further errors.
Triaging Additional Considerations
Connection Pool Configuration: Ensure your connection pool is configured for minimal downtime and efficient operation.
Schema Versioning: Implement a robust schema versioning strategy to minimize the impact of schema changes on running applications.

**Code:**

```sql
-- Example of invalidating a prepared statement
DEALLOCATE PREPARED_STATEMENT_NAME;

-- Example of deallocating all prepared statements
DEALLOCATE ALL;
```

**Flow:**

1. Yes
2. No
3. Identify Affected PreparedStatements
4. Is the issue specific to a fewstatements?
5. Invalidate Specific PreparedStatements
6. Reset Connection Pool
7. Monitor Application Logs forErrors
8. Restart Database or Usepgbouncer Tool
9. Ensure No Residual Issues

