---
type: "practice"
difficulty: "Expert"
---

# Design the migration path from a legacy Oracle system that runs both OLTP and monster reporting on one box, to a split architecture — how do you sequence it to avoid a big-bang cutover risk?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a migration path from a legacy Oracle system that handles both Online Transaction Processing (OLTP) and massive reporting queries, ensuring a smooth transition without a high-risk big-bang cutover. The key is to carefully sequence the migration to minimize downtime and ensure data consistency.

#### Triaging the Question

The interviewer is probing our understanding of OLTP vs. OLAP architecture principles, database partitioning strategies, and change management practices. We need to consider:

How to balance transactional integrity with reporting needs.
The impact on performance and data availability during migration.
Strategies for minimizing downtime.

#### First-Principles Solution
Assess Current Infrastructure: Evaluate the current OLTP workload and identify key performance bottlenecks.
Identify Reporting Needs: Analyze the nature of the reporting queries, including frequency, volume, and criticality.
Plan a Staged Migration:
Step 1: Initial Setup
Set up new databases for OLTP and OLAP.
Implement replication or incremental backups to maintain data consistency between systems.
Step 2: Pilot Phase
Migrate non-critical reporting queries.
Monitor performance and adjust as necessary.
Step 3: Full Migration of Reporting Queries
Gradually migrate all critical reporting queries.
Ensure data integrity through incremental updates or scheduled refreshes.
Step 4: Transition to New Architecture
Decommission old system while maintaining failover capabilities.
Detailed Steps

Initial Setup

Create a new Oracle database for OLTP, ensuring it is optimized for transactional workloads.
Create another database for OLAP with features like materialized views and pre-aggregated data to handle reporting queries more efficiently.
Implement real-time or periodic replication from the OLTP database to the OLAP database.

Pilot Phase

Begin by migrating non-critical reports, such as daily or weekly summaries.
Monitor performance metrics closely to ensure that the new architecture can handle these workloads without significant degradation.
Use tools like Oracle GoldenGate for real-time replication if needed.

Full Migration of Reporting Queries

Migrate critical reporting queries one by one over a period, ensuring each migration is thoroughly tested before going live.
For complex reports, consider breaking them down into smaller components that can be scheduled independently to reduce the overall impact on the system.
Implement data consistency checks between the OLTP and OLAP databases during this phase.

Transition to New Architecture

Once all reporting queries are migrated, decommission the old Oracle system.
Ensure there is a failover mechanism in place for critical transactions that might still require access to the legacy system temporarily.
Perform final health checks and performance optimizations before fully switching off the old environment.

#### Closing Summary

The key to successfully migrating from a monolithic OLTP + reporting architecture to a split OLTP + OLAP architecture is careful planning, incremental execution, and robust testing. By following this staged approach, we can minimize downtime, ensure data integrity, and optimize performance for both transactional and analytical workloads.

#### Common Mistakes/Follow-Ups
Avoiding Incremental Migrations: Failing to break down the migration into smaller steps can lead to high-risk big-bang cutover.
Ignoring Data Consistency: Not implementing robust data synchronization mechanisms between OLTP and OLAP databases can result in inconsistencies.
Overlooking Performance Monitoring: Insufficient monitoring during the transition phase may miss critical performance issues that could impact user experience.

**Comparison:**

```
region	product	total_sales
North	A	100
```

**Code:**

```sql
-- Example of a materialized view creation for OLAP database
CREATE MATERIALIZED VIEW sales_summary_mview AS
SELECT region, product, SUM(sales) AS total_sales
FROM sales_table
GROUP BY region, product;
```

**Example input / output:**

```
Sample Input:
JSON
Copy
{
  "region": "North",
  "product": "A"
}
Output:
JSON
Copy
[
  {"region": "North", "product": "A", "total_sales": 100}
]
```

**Data:**

```
Schema: sales_table (region, product, sales)
Sample Data:
JSON
Copy
[
  {"region": "North", "product": "A", "sales": 100},
  {"region": "South", "product": "B", "sales": 200},
  ...
]
```

**Flow:**

1. Initial Setup
2. Replication/IncrementalBackup
3. OLTP Database
4. OLAP Database
5. Pilot Phase - Non-CriticalReports
6. Migrate Critical Queries Oneby One
7. Transition to New Architecture
8. Failsafe Mechanism for LegacySystem

