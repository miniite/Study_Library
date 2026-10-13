---
type: "practice"
difficulty: "Basic"
---

# What is the default Time Travel retention period and how can it be changed?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Time Travel and Storage Costs](<../../../../TOOLS/Snowflake/Snowflake_Platform/Time_Travel_and_Storage_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Snowflake's Time Travel feature allows you to query historical states of your tables, which is incredibly useful for debugging or ensuring data consistency in data lakes. The default retention period for Time Travel is 90 days, but this can be adjusted based on the specific needs of your organization.

#### Triaging the Question

The question is essentially asking two things:

What is the default Time Travel retention period?
How can this default period be changed?

#### Assumptions
and Clarifications:
We are focusing on a Snowflake Data Warehouse environment.
The term "retention" refers to the number of days Snowflake retains historical data versions for Time Travel.
Detailed Solution from First Principles:

1. Understanding Default Retention: By default, Snowflake sets the retention period for Time Travel to 90 days. This means that when you perform a query using SELECT * FROM TABLE AS OF '2023-06-01', Snowflake will return data as it was on June 1st, but only if this version of the data is still retained within these 90 days.

2. Changing the Retention Period: To change the default retention period, you can use the ALTER WAREHOUSE command to modify the warehouse's settings. Specifically, you would set the TIME_TRAVEL_RETENTION_IN_DAYS parameter. Here’s an example of how this is done:

SQL
ALTER WAREHOUSE my_warehouse SET TIME_TRAVEL_RETENTION_IN_DAYS = 180;

This command changes the retention period for the my_warehouse to 180 days.

3. Practical Considerations:

Data Volume: Longer retention periods require more storage space and can impact performance, as Snowflake needs to store multiple versions of your data.
Cost Implications: Longer retention periods will also increase costs due to the additional storage required.
Regulatory Requirements: Depending on your industry (e.g., healthcare, finance), regulatory compliance might dictate a minimum or maximum retention period.

#### Closing Summary
and Common Mistakes/Follow-Ups:

In summary, the default Time Travel retention in Snowflake is 90 days. To change this, you use the ALTER WAREHOUSE command with the TIME_TRAVEL_RETENTION_IN_DAYS parameter. It's crucial to balance the needs of data integrity against storage and cost considerations.

#### Common mistakes
include not considering the storage implications or neglecting to update warehouses that may have different retention periods set by default. Always ensure you understand the current settings before making changes, especially in production environments.

For follow-ups, one might ask about best practices for managing Time Travel retention or how to monitor storage usage related to historical data.

**Code:**

```sql
ALTER WAREHOUSE my_warehouse SET TIME_TRAVEL_RETENTION_IN_DAYS = 180;
```

