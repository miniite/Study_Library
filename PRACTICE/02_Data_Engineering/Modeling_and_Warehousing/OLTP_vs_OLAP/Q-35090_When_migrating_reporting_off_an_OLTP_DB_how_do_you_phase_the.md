---
type: "practice"
difficulty: "Deep"
---

# When migrating reporting off an OLTP DB, how do you phase the cutover (dual-run, shadow reads, gradual traffic shift) so you can prove correctness and roll back safely?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When migrating reporting from an OLTP database to a more suited OLAP solution, ensuring a seamless transition is crucial. The key phases involve setting up a dual-run environment (or shadow reads) followed by gradual traffic shift. This approach allows you to verify the correctness of the new system and ensure it can handle the load without disrupting operations. Here’s a detailed breakdown:

#### Triage

The primary goal is to minimize downtime and risk during the migration process. The interviewer is probing how you balance the transition between the old OLTP database (source) and the new OLAP environment (target). Key assumptions include that both systems are initially functional, and there's sufficient infrastructure for a dual-run period.

#### First Principles Solution
Initial Setup: Create a shadow environment with the target OLAP schema.
Data Synchronization: Periodically synchronize data from the OLTP to the OLAP system using ETL processes.
Dual-Run Phase:
Shadow Reads: Perform initial reporting on both systems and compare results.
Parallel Processing: Gradually shift reporting queries from the OLTP to the OLAP system while continuously monitoring for discrepancies.
Validation and Testing: Conduct extensive testing, including load testing, stress testing, and sanity checks.
Rollback Strategy: Prepare a rollback plan in case the new system fails validation.
Detailed Steps
Initial Setup:
Design the OLAP schema to match reporting requirements (aggregated tables, summarized data).
Set up ETL processes to regularly transfer fresh data from OLTP to OLAP.
Data Synchronization:
Use incremental or full batch ETL strategies based on the volume and frequency of changes in the OLTP system.
Dual-Run Phase:
Start with a small subset of reporting queries and gradually increase the load.
Monitor real-time differences between the results from both systems.
Validation and Testing:
Ensure consistency by comparing results for critical reports.
Test edge cases such as large datasets, complex queries, and time-sensitive data.
Rollback Strategy:
Have a clear plan to revert changes if issues arise during validation.
Maintain a backup of the original OLTP environment.

#### Closing Summary

By following this phased approach, you can ensure that the migration is smooth and risk-free. The dual-run phase allows thorough testing before fully committing to the new system. Common mistakes include insufficient testing, lack of a rollback plan, or failing to manage data consistency adequately during the transition.

#### Common Mistakes/Follow-Ups
Common Mistake: Skipping extensive validation due to time constraints.

#### Follow-Up
Discuss how to optimize ETL processes and automate tests to reduce manual effort.

**Comparison:**

| Step | Action                         | Key Considerations            |
|------|--------------------------------|------------------------------|
| 1    | Initial Setup                  | Match schema, prepare ETL     |
| 2    | Data Synchronization           | Incremental/full batch        |
| 3    | Dual-Run Phase                 | Shadow reads, gradual shift   |
| 4    | Validation and Testing         | Comprehensive, edge cases     |
| 5    | Rollback Strategy              | Clear plan, backup            |

**Code:**

```sql
-- Example SQL for an incremental ETL process
INSERT INTO olap_table (SELECT * FROM oltp_table WHERE last_update_time > (SELECT MAX(last_update_time) FROM olap_table));
```

**Example input / output:**

```
Sample Input/Output:
PLAINTEXT
Copy
Input: Run a report on monthly sales from the OLTP system.
Output: Compare results with the same query in the OLAP system to ensure consistency.
```

**Data:**

```
Sample Data Schema:
YAML
Copy
OLTP Table: customer_orders
  - order_id
  - customer_id
  - product_id
  - quantity
  - order_date

OLAP Table: monthly_sales
  - month
  - total_sales
```

**Flow:**

1. Initial Setup
2. Synchronize Data
3. Dual-Run Phase
4. Validation and Testing
5. Rollback Strategy
6. Migrate Fully
7. Monitor Post-Migration

