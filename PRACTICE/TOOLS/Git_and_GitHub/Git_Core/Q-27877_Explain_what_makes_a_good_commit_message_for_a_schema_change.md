---
type: "practice"
difficulty: "Basic"
---

# Explain what makes a good commit message for a schema change beyond the summary line, including why capturing the operational risk (lock behavior, backfill duration, rollback plan) in the body pays off during a future incident bisect.

**Difficulty:** Basic · **Levels:** SDE-1, SDE-1/2 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A good commit message for a schema change should be clear and detailed to aid in understanding, tracking, and troubleshooting changes. Beyond the summary line, the body of the commit message is crucial as it captures operational risks such as lock behavior, backfill duration, and rollback plans. This information pays off during future incidents when diagnosing issues or performing an "incident bisect."

When making a schema change, consider the following aspects to ensure your commit message is effective:

Lock Behavior: Describe any locks that are being acquired or released by the migration process. Locks can cause deadlocks and performance issues if not managed properly.
Backfill Duration: Estimate the time it will take for the backfill operation to complete. This helps in planning maintenance windows and managing user expectations.
Rollback Plan: Outline a detailed rollback strategy, including steps that need to be taken if something goes wrong during or after the migration.

By capturing these details, you make future diagnostics easier and reduce potential downtime.

**Code:**

```plaintext
# Example Commit Message for Schema Change

## Summary Line:
Add new columns to users table to track user activity

## Body:
### Lock Behavior:
- Acquire write locks on the `users` table during schema changes to prevent read/write conflicts.
- Release locks within 15 minutes to avoid deadlocks and performance issues.

### Backfill Duration:
- Estimated backfill time: 2 hours. This includes initial data population and verification steps.

### Rollback Plan:
- Backup current state of the database before making any changes.
- If migration fails, revert to a known good state using the backup.
- Manually remove newly added columns and restore original schema structure.
```

**Flow:**

1. Yes
2. Yes
3. Yes
4. Schema Change
5. Does the commit messagecapture lock behavior?
6. Detailed Lock Behavior
7. Is backfill duration estimatedand captured?
8. Backfill Duration Estimation
9. Rollback plan documented?
10. Commit Message Complete

