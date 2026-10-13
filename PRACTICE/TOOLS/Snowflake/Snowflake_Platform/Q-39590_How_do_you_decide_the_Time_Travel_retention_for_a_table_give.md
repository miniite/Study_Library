---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide the Time Travel retention for a table given both cost pressure and the SLA for accidental-delete recovery?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Time Travel and Storage Costs](<../../../../TOOLS/Snowflake/Snowflake_Platform/Time_Travel_and_Storage_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding on the Time Travel retention period for a Snowflake table, as a Data Engineer with intermediate experience, you need to balance both cost pressures and service level agreements (SLAs) for accidental delete recovery. This involves understanding how long you must retain data to meet your SLA while minimizing costs.

To triage this problem, let's break down the key components:

Understanding Time Travel in Snowflake: Time Travel allows you to recover deleted records within a specified retention period.
Cost Pressure: Minimizing storage costs by reducing the retention period can be crucial for budget management.
SLA for Accidental Delete Recovery: The SLA dictates how long data needs to be retained to ensure compliance with business and regulatory requirements.

Now, let's walk through a step-by-step first-principles solution:

Identify the SLA Requirements:

Determine the maximum allowable time for accidental deletion recovery based on your organization’s policies and regulatory requirements.
For example, if you need to recover data within 30 days in case of an accidental delete, set the retention period accordingly.

Cost Analysis:

Calculate the cost impact of different retention periods using Snowflake's pricing model. This includes storage costs for each day of retention.
Use a cost-benefit analysis to determine the optimal balance between recovery needs and cost efficiency.

Risk Assessment:

Evaluate the risk associated with retaining data too long or too short. Over-retention can lead to unnecessary costs, while under-retention risks violating SLAs.

#### Cons
ider the likelihood of accidental deletions and their impact on business operations.

Setting the Retention Period:

Based on the above analysis, set a retention period that meets your SLA requirements without incurring excessive costs.
For instance, if you need 30 days for recovery but can manage with 15-20 days due to lower risk of accidental deletions and less stringent compliance needs.

Monitoring and Optimization:

Continuously monitor the usage patterns and retention policies to ensure they remain optimal as business requirements evolve.
Regularly review costs and SLA compliance, making adjustments as necessary.

By following these steps, you can make an informed decision that balances cost efficiency with data recovery needs.

**Comparison:**

```
Retention Period	Cost Impact ($/day)	Compliance Risk
30 days	$0.75	Low
21 days	$0.525	Medium
14 days	$0.35	High
```

**Code:**

```sql
-- Example Snowflake SQL to set Time Travel retention
ALTER TABLE my_table SET RETENTION 30;
```

**Example input / output:**

```text
Input:
- SLA requirement: recover data within 30 days after an accidental delete.
- Cost analysis of storage cost per day: $0.025.

Output:
- Decision: Set Time Travel retention to 21 days (a balance between meeting the SLA and minimizing costs).
```

**Data:**

```text
- **Sample Retention Periods:**
    - SLA Requirement: 30 days for accidental delete recovery.
    - Cost Analysis: Storage cost per day = $0.025.
```

**Flow:**

1. Identify SLA Requirements
2. Set Retention Period
3. Monitor Usage Patterns
4. Maintain Optimal Balance
5. Cost Analysis
6. Optimize Costs

