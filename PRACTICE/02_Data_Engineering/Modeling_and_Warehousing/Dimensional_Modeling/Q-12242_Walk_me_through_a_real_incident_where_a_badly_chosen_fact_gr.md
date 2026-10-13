---
type: "practice"
difficulty: "Deep"
---

# Walk me through a real incident where a badly chosen fact grain caused wrong numbers in a dashboard, and how you diagnosed and fixed it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In one instance, we encountered a critical issue with our financial reporting dashboard that was due to an improperly chosen fact grain. The problem manifested as incorrect total sales figures across multiple regions, leading to significant discrepancies in revenue forecasts and impacting budgeting processes.

The question the interviewer is probing here involves understanding:

The concept of fact grains in data modeling.
The impact of improper fact grain selection on reporting accuracy.
The process of diagnosing such issues and implementing corrective measures.

To diagnose this issue, we first needed to understand that the fact grain (the level at which the fact table is aggregated) was chosen too coarsely. In this case, it aggregated sales data by week instead of day. This led to miscalculations in regional totals because daily fluctuations were not properly captured.

Step-by-Step Diagnosis and Fix

Identify the Problem:

The dashboard showed incorrect total sales figures for several regions.
Suspected that this was due to an improperly chosen fact grain.

Review Fact Table Aggregation Level:

We examined the fact table schema, which aggregated sales data by week.
Realized that daily fluctuations in sales were not accurately captured, leading to discrepancies when summing up weekly totals.

Reaggregate Data to Appropriate Granularity:

Decided to change the fact grain from weekly to daily aggregation.
This would allow us to capture more granular data and ensure accurate reporting at all levels of detail.

Verify Correctness with Test Cases:

Ran test cases across different regions and time periods to validate that the new daily aggregation was correctly summing up to the regional totals.
Ensured that the new model provided consistent and accurate results compared to historical data.

Implement New Model in Production:

Migrated the updated fact table to the production environment.
Performed a thorough validation by comparing old and new dashboards side-by-side for several weeks to ensure no discrepancies existed.

Communicate Changes to Stakeholders:

Provided training sessions for stakeholders on how to interpret data with the improved granularity.
Ensured that all relevant teams understood the implications of the changes, including potential impacts on forecasting and budgeting processes.

#### Closing Summary

By diagnosing and fixing this issue, we were able to ensure accurate financial reporting across multiple regions. The key lessons learned from this experience are:

Choosing the appropriate fact grain is crucial for maintaining data integrity.
Regularly reviewing and validating aggregation levels can prevent major discrepancies in reports.
Communication with stakeholders is essential when implementing changes that affect critical business processes.

#### Common Mistakes
and Follow-Ups

Common Mistake:

Failing to recognize the impact of improper fact grain selection on reporting accuracy.

#### Follow-Up
Questions:

How do you handle data inconsistencies during the migration process?
What steps would you take if the issue persisted after the initial fix?

**Comparison:**

```
table "sales_fact" {
  region_id: INT,
  date_week_start: DATE,
  total_sales_weekly: BIGINT,
  date_day: DATE,
  total_sales_daily: BIGINT
}
```

**Code:**

```sql
-- Example SQL for changing fact table aggregation from weekly to daily

ALTER TABLE sales_fact
MODIFY COLUMN total_sales_weekly BIGINT,
ADD COLUMN total_sales_daily BIGINT;

UPDATE sales_fact SET total_sales_daily = SUM(total_sales) OVER (PARTITION BY region_id, date_day);
```

**Example input / output:**

```text
Input:
- Original weekly aggregation results:
  - Region A: $10,000 (weekly)

Output after daily aggregation fix:
- Corrected daily aggregation results:
  - Region A: [$9800, $9950, ..., $10200] -> Total = $10,000
```

**Data:**

```json
{
  "sales_fact": {
    "columns": [
      {"name": "region_id", "type": "INT"},
      {"name": "date_week_start", "type": "DATE"},
      {"name": "total_sales_weekly", "type": "BIGINT"},
      {"name": "date_day", "type": "DATE"},
      {"name": "total_sales_daily", "type": "BIGINT"}
    ]
  }
}
```

**Flow:**

1. Identify incorrect grain
2. Review fact table schema
3. Reaggregate to daily level
4. Migrate to production
5. Validate results
6. Communicate changes
7. Train stakeholders
8. No issues
9. Continue monitoring

