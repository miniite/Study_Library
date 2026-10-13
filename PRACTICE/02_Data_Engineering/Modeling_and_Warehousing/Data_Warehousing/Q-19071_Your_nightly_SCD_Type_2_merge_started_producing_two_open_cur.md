---
type: "practice"
difficulty: "Deep"
---

# Your nightly SCD Type 2 merge started producing two open (current) rows per business key. How would you debug and fix the root cause?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering an issue where a nightly SCD Type 2 merge produces two open (current) rows per business key, it's crucial to methodically identify and resolve the root cause. This problem can significantly impact data integrity and accuracy in your data warehouse. We'll break down the debugging process step by step.

Firstly, let's clarify what we're dealing with: SCD Type 2 (Slowly Changing Dimension Type 2) is used when a dimension table needs to retain historical records while also updating them as necessary. The primary issue here is that two rows are being marked as current for the same business key, which should ideally have one current row at any given time.

Triaging the Issue

The question is essentially asking how to systematically debug and fix the root cause of this anomaly. Here's what we need to probe:

Understand the Merge Logic: Review the merge logic used in your ETL process to ensure that it correctly identifies new, changed, or deleted records.
Check Data Sources: Ensure that both source systems and incremental updates are accurately sending the correct data.
Validate Business Rules: Verify if there were any recent business rule changes that might have led to this unexpected behavior.
Audit Logs: Inspect ETL logs for any unusual patterns or errors during the merge process.
Debugging Step-by-Step

Identify Affected Rows:

Query your data warehouse to find all rows with duplicate current flags for a given business key.
SQL
SELECT business_key, current_flag, COUNT(*)
FROM your_dimension_table
GROUP BY business_key, current_flag
HAVING COUNT(*) > 1;

Review Source Data:

Compare the source data from both systems (let's call them System A and System B) to see if there are any discrepancies.
SQL
-- Example query for comparing records in two sources
SELECT * FROM system_a_source WHERE business_key = 'some_key'
INTERSECT
SELECT * FROM system_b_source WHERE business_key = 'some_key';

Validate Business Logic:

Check the logic that determines whether a row should be marked as current or not.
SQL
-- Example query to inspect business logic for marking rows as current
SELECT * FROM your_dimension_table WHERE business_key = 'some_key' AND current_flag = 'Y';

Check ETL Process:

Ensure the ETL process is correctly handling updates and inserts.
SQL
-- Example query to inspect ETL processes for a specific business key
SELECT * FROM your_dimension_table WHERE business_key = 'some_key' ORDER BY load_timestamp DESC;

Audit Logs:

Examine the logs from your ETL process for any errors or warnings that could indicate issues with data handling.
PLAINTEXT
-- Example log output
2023-10-01T00:00:00Z | Warning: Duplicate key detected during merge for business_key X

#### First-Principles Solution

Given the above steps, here’s a more detailed approach to fixing the issue:

Audit and Update Business Logic:
If the logic is flawed, update it to ensure that only one row per business key can be marked as current.
Validate Data Sources:
Ensure both sources are providing consistent data by performing cross-checks and resolving any discrepancies.
Improve ETL Process:
Enhance your ETL process to handle edge cases more robustly, such as ensuring proper handling of null values or duplicates.

#### Closing Summary

By methodically working through the potential causes and validating each component of your data pipeline, you can identify and resolve the root cause of the issue. This process not only fixes the current problem but also ensures that future merges are handled correctly.

#### Common Mistakes/Follow-Ups

#### Common Mistakes
Failing to thoroughly inspect both source systems, misinterpreting business rules, or overlooking critical audit logs.

#### Follow-Ups
Discussing long-term solutions such as implementing more robust ETL validation and testing processes, or considering automated tools for better consistency.

**Comparison:**

| Business Key | Current Flag |
|--------------|-------------|
| 12345        | Y           |
| 12345        | Y           |

**Code:**

```sql
-- Example SQL queries
SELECT business_key, current_flag, COUNT(*)
FROM your_dimension_table
GROUP BY business_key, current_flag
HAVING COUNT(*) > 1;

SELECT * FROM system_a_source WHERE business_key = 'some_key'
INTERSECT
SELECT * FROM system_b_source WHERE business_key = 'some_key';

SELECT * FROM your_dimension_table WHERE business_key = 'some_key' AND current_flag = 'Y';
```

**Example input / output:**

```text
Input:
Business Key: 12345

Expected Output:
No duplicate current rows found.

Actual Output:
Two rows with current_flag = 'Y' for business_key 12345.
```

**Data:**

```plaintext
-- Sample log entries for debugging
2023-10-01T00:00:00Z | Warning: Duplicate key detected during merge for business_key X
```

**Flow:**

1. Yes
2. No
3. Start Debugging
4. Identify Affected Rows
5. Query Data Warehouse
6. Review Source Data
7. Validate Business Logic
8. Check ETL Process
9. Audit Logs
10. Update Business Logic
11. Fix and Test
12. End Debugging

