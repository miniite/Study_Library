---
type: "practice"
difficulty: "Basic"
---

# What is the difference between deleting a row and updating a status column to mark it inactive?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DELETE TRUNCATE and DROP](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL, you might need to manage the lifecycle of data by either completely removing rows or marking them as inactive by updating their status. The core difference lies in how these operations affect your database and potential future queries.

#### Triaging the Question

The question asks about two common methods to handle "deleting" a row: outright deletion using DELETE or soft-deletion by changing a column value, such as setting an inactive flag. The interviewer is probing understanding of data integrity, performance implications, and the trade-offs between these approaches.

#### First Principles Solution

Deletion (DELETE Statement):

What it Does: Permanently removes the row from the table.
When to Use It: When a record should no longer exist in the database and you don't need any historical data about its status or last updates.
Impact on Indexes: Deletes the entry, so indexes are updated, which can impact performance if the table is large.

#### Pros
Simple to implement.
No risk of accidentally retaining inactive records.
Faster than updating because it doesn't require locking other rows or writing extra data.

Soft-Deletion (Updating a Status Column):

What it Does: Marks the row as inactive by changing a specific column, usually an is_active field, to false or another suitable value.
When to Use It: When you need to retain historical data and may want to reactivate rows in the future. This approach also helps with audit trails and can be useful for compliance reasons.
Impact on Indexes: Only changes a single column's value, so it doesn't impact indexes unless the is_active field is indexed.

#### Pros
Retains historical data, which might be useful for reporting or compliance.
Can be queried easily to find inactive records.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, choosing between DELETE and soft-deletion depends on your specific use case. Soft-deletion can lead to cleaner audit trails but requires additional logic to handle reactivation. Deletion is simpler and faster but results in permanent data loss.

#### Common Mistakes

Failing to consider the need for historical data when using deletion.
Not properly setting up triggers or constraints with soft-deletion, leading to inconsistencies.

#### Follow-Up
Discussion Points
If you're concerned about performance, discuss how indexing can be used to optimize either approach.
Talk about the implications of cascading operations and referential integrity rules in both cases.
Discuss scenarios where row versioning might be more appropriate than soft-deletion or deletion.

