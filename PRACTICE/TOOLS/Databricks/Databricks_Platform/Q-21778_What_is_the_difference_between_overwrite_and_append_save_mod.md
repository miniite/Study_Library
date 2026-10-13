---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between overwrite and append save modes when writing a table and when would you use each?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Query Performance](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Query_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Databricks, understanding the differences between overwrite and append save modes is crucial for managing data in Delta Tables or DataFrames. These save modes determine how your write operations affect existing data, which can impact storage efficiency and integrity.

#### Triage

The question asks to differentiate between overwrite and append save modes when writing a table using Databricks. The interviewer might be probing your understanding of these fundamental concepts in data management and their implications on data consistency and performance. I'll assume the context is working with Delta Tables, as they are widely used in Databricks.

#### First-Principles Solution
Overwrite Mode:
Behavior: This mode overwrites the entire target table with the new data.
Use Case: Ideal for scenarios where you want to completely replace the existing data with fresh data. It ensures that no previous records are retained unless explicitly merged.

#### Example Scenario
If you're performing a full reload of your dataset, overwrite is appropriate.
Append Mode:
Behavior: This mode appends new rows to the target table while keeping all existing data intact.
Use Case: Suitable for incremental or real-time updates where only additional records need to be added without affecting existing ones.

#### Example Scenario
When you have a continuous stream of new data that needs to be added to an existing dataset, append is the right choice.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, choose overwrite when you want to start fresh with your data, replacing all existing records. Use append for incremental updates where only new rows should be added. Both modes have their own advantages and use cases depending on the nature of your dataset and the operation you're performing.

#### Common mistakes
include using overwrite in situations that require partial updates or when you need to preserve historical data, leading to loss of information. Conversely, using append for full reloads could lead to an unnecessarily large table as old records are not removed.

#### Follow-ups
might involve discussing performance implications and best practices for managing data with Databricks, such as using partitioning and clustering for better query optimization.

**Code:**

```python
# Example of overwrite mode in Databricks
overwrite_df.write.format("delta").mode("overwrite").save("/path/to/table")

# Example of append mode in Databricks
append_df.write.format("delta").mode("append").save("/path/to/table")
```

