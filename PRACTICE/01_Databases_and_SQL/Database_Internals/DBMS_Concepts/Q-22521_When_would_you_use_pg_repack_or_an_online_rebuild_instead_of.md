---
type: "practice"
difficulty: "Expert"
---

# When would you use pg_repack or an online rebuild instead of VACUUM FULL to reclaim space without a long exclusive lock?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Batching Writes](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Batching_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address when pg_repack or an online rebuild should be used over VACUUM FULL, we need to understand the context and trade-offs involved in each approach. VACUUM FULL is a powerful command that reclaims dead space by performing a full table rewrite, but it requires an exclusive lock on the table for the duration of its execution, which can cause significant disruptions during database operations. On the other hand, pg_repack and online rebuilds are designed to reclaim space more efficiently with minimal disruption.

#### Triage
The Question

The question is essentially asking under what conditions or scenarios one would prefer using pg_repack or an online rebuild (ALTER TABLE ... REBUILD) over a VACUUM FULL. These methods all aim to reclaim space in the database, but they differ significantly in their impact on performance and system availability.

#### Assumptions
The database is PostgreSQL.
We are dealing with large tables that require regular maintenance for space optimization.
The goal is to minimize downtime and maintain high levels of service during maintenance.

#### First Principles Solution
VACUUM FULL

VACUUM FULL works by scanning the table, marking the dead rows as free space, then rewriting all the live data. This process requires an exclusive lock on the table for the entire duration, making it unsuitable in environments where high availability is critical.

pg_repack

pg_repack uses a background worker to move rows from one page to another while minimizing locks and locking overheads. It can be used without causing significant downtime but requires careful planning due to its complexity and potential impact on performance during the process.

Online Rebuild

An ALTER TABLE ... REBUILD operation is essentially a VACUUM FULL that can be executed online, meaning it does not require an exclusive lock on the table. However, it still requires some downtime for the metadata changes to take effect.

When to Use Each Method

pg_repack:

When: High availability is critical and you need a tool that can perform space reclamation while minimizing locks.
Why: pg_repack minimizes the impact on other database operations by using background workers, making it suitable for live environments where downtime must be kept to an absolute minimum.

Online Rebuild:

When: You have a table that is not heavily accessed during maintenance windows and you need a simple way to reclaim space without locking.
Why: The ALTER TABLE ... REBUILD command provides a straightforward approach for reclaiming space, but it still requires some downtime due to metadata updates.

VACUUM FULL:

When: You are willing to tolerate significant downtime during maintenance and the table size is such that VACUUM FULL can complete in an acceptable amount of time.
Why: VACUUM FULL is a powerful tool for space reclamation, but it’s not recommended in environments where downtime must be minimized.

#### Closing Summary

In summary, when deciding between pg_repack, online rebuilds (ALTER TABLE ... REBUILD), and VACUUM FULL, consider the criticality of your database's uptime. For highly available systems, pg_repack is often the best choice due to its minimal locking requirements. Online rebuilds provide a simpler alternative that still requires some downtime but can be executed quickly if necessary.

#### Common Mistakes/Follow-Ups
Misunderstanding the Impact: Failing to recognize the differences in how each method affects system availability and performance.
Choosing the Wrong Tool: Selecting VACUUM FULL when a more granular approach is needed, or opting for an online rebuild without adequate downtime planning.

**Code:**

```

-- Example of using pg_repack SELECT repack.repack_table('schemaname.tablename');

CODE
Copy
```sql
-- Example of using ALTER TABLE ... REBUILD
ALTER TABLE schemename.tablename REBUILD;
```

