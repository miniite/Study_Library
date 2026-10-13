---
type: "practice"
difficulty: "Deep"
---

# You need to reclaim space from a bloated table without an exclusive lock that VACUUM FULL takes; explain how pg_repack works online and what edge cases (triggers, exclusion constraints) make it risky.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Retention Purging at Scale](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Retention_Purging_at_Scale.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the need to reclaim space from a bloated table without an exclusive lock, PostgreSQL's pg_repack tool can be used in an online mode. It allows you to shrink or reorganize the table structure while keeping the database available for ongoing transactions. However, it has certain limitations and edge cases that make its usage somewhat risky.

#### Triaging the Question

The core of the question is about understanding pg_repack and identifying potential pitfalls related to triggers and exclusion constraints. The interviewer is probing your knowledge of PostgreSQL's internal mechanisms and your ability to manage risks in a production environment.

#### First-Principles Solution

pg_repack works by moving rows around in place, which means it doesn't require an exclusive lock on the table, unlike VACUUM FULL. This makes it suitable for environments where downtime is not acceptable. However, certain database structures and triggers can interfere with its operation.

Understanding pg_repack
Operation: pg_repack works by reading rows from the source table, sorting them if necessary, and writing them back to a new location.
Concurrency: It uses an MVCC (Multi-Version Concurrency Control) approach, allowing concurrent transactions to read the old data while writes are being applied.
Key Edge Cases
Triggers:
Triggers can be problematic because they may not handle the row movement correctly if the triggers are defined on certain columns or involve complex logic.
Exclusion Constraints:
Exclusion constraints can cause issues when rows are moved around, as they need to be checked and possibly adjusted during the reorganization process.
Detailed Solution

Triggers:

Triggers that modify the row being updated or inserted can fail if the trigger logic expects the original state of the row.
For example, a trigger that updates another table based on changes in the current row might not work correctly during pg_repack because the row's state is temporarily altered.

Exclusion Constraints:

Exclusion constraints ensure that rows with conflicting values do not violate defined rules. During reorganization, these constraints must be maintained.
If a row needs to move from one place to another, the exclusion constraint check might need to be run multiple times, which can lead to performance issues or even failures if the constraint is complex.

#### Common Mistakes
and Follow-Ups
Assuming No Impact: One common mistake is assuming that triggers and constraints will handle reorganization seamlessly without any additional configuration.
Overlooking Performance Impacts: The operation might still cause performance degradation, especially on large tables with complex constraints or many triggers.
Ignoring Cleanup: After pg_repack completes, you should verify that all data integrity checks are satisfied and the database is in a consistent state.

#### Example
of Edge Case Handling
SQL
-- Suppose we have a table with an exclusion constraint and some triggers
CREATE TABLE my_table (
    id SERIAL PRIMARY KEY,
    value INT NOT NULL CHECK (value > 0),

#### CONS
TRAINT excl_value_exclusive EXCLUDE USING gist (value WITH =
);

CREATE TRIGGER check_value BEFORE INSERT ON my_table FOR EACH ROW EXECUTE FUNCTION raise_exception_if_value_is_not_positive();
The exclusion constraint excl_value_exclusive ensures that no two rows have the same value.
If you use pg_repack on this table, it might need to temporarily violate or handle these constraints more carefully.

#### Closing Summary

In summary, while pg_repack is a powerful tool for online schema management in PostgreSQL without requiring exclusive locks, it comes with certain risks. Triggers and exclusion constraints can cause issues that require careful handling. Understanding the underlying mechanisms and potential pitfalls is crucial to effectively use this tool in production environments.

Self-Check

The solution covers all aspects of pg_repack usage, edge cases, and common mistakes. All key points are addressed with examples.

**Code:**

```sql
-- Example SQL for table creation with constraints and triggers
CREATE TABLE my_table (
    id SERIAL PRIMARY KEY,
    value INT NOT NULL CHECK (value > 0),
    CONSTRAINT excl_value_exclusive EXCLUDE USING gist (value WITH =
);

CREATE TRIGGER check_value BEFORE INSERT ON my_table FOR EACH ROW EXECUTE FUNCTION raise_exception_if_value_is_not_positive();
```

