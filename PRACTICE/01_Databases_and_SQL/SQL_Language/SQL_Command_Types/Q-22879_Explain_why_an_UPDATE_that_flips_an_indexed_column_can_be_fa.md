---
type: "practice"
difficulty: "Deep"
---

# Explain why an UPDATE that flips an indexed column can be far slower than one that doesn't, and how you'd measure the write amplification.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Online Schema Changes](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing database operations, the performance of an UPDATE statement can vary significantly based on whether the column being updated is indexed or not. An UPDATE that changes a value in an indexed column often incurs additional overhead because the database needs to update the index as well. This extra work can make such updates much slower than those that do not affect indexed columns.

To fully understand this, let's break it down from first principles and explore how you would measure write amplification (the amount of writes caused by a single data change).

#### Triage
What the Question is Asking

The question asks us to explain why an UPDATE statement that changes an indexed column can be significantly slower than one that doesn’t, and to discuss how we would measure this write amplification. The core of the question revolves around understanding index maintenance overhead and the impact on database performance.

#### Assumptions
and Key Concepts
Indexes: These are data structures that improve query performance by providing quick access to rows.
Index Maintenance Overhead: Updating a table also requires updating the associated indexes, which can be costly in terms of time and resources.
Write Amplification: This is the ratio of write operations performed on storage due to an update operation.

#### First-Principles Solution
Why Indexed Columns Can Slow Down Updates

When you execute an UPDATE statement that changes a value in an indexed column, the database must perform additional work:

Updating the Index: The old index entry for the row being updated needs to be invalidated and replaced with the new one.
Maintaining ACID Properties: Ensure transactional integrity by logging the change properly.

These operations can significantly increase the time taken for the UPDATE statement, especially in highly indexed tables or when dealing with large datasets.

Measuring Write Amplification

To measure write amplification, you need to track both the number of data writes and index updates. A common approach is to use a performance monitoring tool that can capture write operations at various levels of the storage hierarchy.

Before the Update: Record the initial state of your database (data size, index size).
Execute the UPDATE Statement.
After the Update: Capture the final state again and compare the differences:
Calculate the number of data writes.
Measure how many additional index entries were updated or invalidated.
Practical Example

Let's say you have a table with an indexed column status, and you execute the following update:

SQL
UPDATE users SET status = 'active' WHERE id = 123;

You would use a database performance monitoring tool to record:

The initial number of rows in the data section.
The initial number of index entries (e.g., B-tree nodes) associated with the status column.
The time taken for the update operation.

After the update, you would note:

The updated row count.
The changed or newly created index entries.
Calculating Write Amplification

Write amplification can be calculated as follows:

PLAINTEXT
Write Amplification = (Total Number of Writes After Update - Initial Number of Data and Index Writes) / Initial Number of Writes

For instance, if the initial write count was 100 (50 data + 50 index), and after the update, you recorded 200 writes (100 data + 100 new indexes), then:

PLAINTEXT
Write Amplification = (200 - 100) / 100 = 1.0

In this example, there was no net increase in write operations due to the update; however, in many real-world scenarios, you might see a higher amplification.

#### Closing Summary
and Common Mistakes

To summarize, updates that change indexed columns can be slower because they necessitate additional index maintenance work. Measuring write amplification involves tracking both data and index writes before and after an UPDATE. Key mistakes to avoid include not accounting for the overhead of index updates or assuming that all updates have negligible side effects.

#### Common Mistakes
Ignoring Index Overhead: Assuming that changes to indexed columns are as fast as those to non-indexed ones.
Misinterpreting Write Metrics: Failing to distinguish between data writes and index updates when calculating write amplification.
Overlooking ACID Compliance: Not understanding the importance of ensuring transactional integrity during the update process.

**Code:**

```sql
-- Example SQL Update Statement
UPDATE users SET status = 'active' WHERE id = 123;
```

