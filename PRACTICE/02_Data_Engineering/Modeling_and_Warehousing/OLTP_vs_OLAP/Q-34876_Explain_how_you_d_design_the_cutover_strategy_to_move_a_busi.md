---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd design the cutover strategy to move a business-critical dashboard from live OLTP to the warehouse with a shadow/dual-run period, and what gates green-light the switch.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a seamless transition of a business-critical dashboard from an operational transactional system (OLTP) to a data warehousing environment (OLAP), we need to carefully design a cutover strategy that includes a dual-run or shadow period. This approach allows us to validate the accuracy and consistency between the live OLTP database and the warehouse, mitigating risks during the actual switch.

The core idea is to ensure both systems operate in tandem for a period of time, allowing us to compare data and identify any discrepancies early on. Once these issues are resolved, we can proceed with confidence to shut down the OLTP system and fully transition to the warehouse.

#### Triage
Understanding the Question

The question asks about designing a cutover strategy that involves moving a critical dashboard from an OLTP environment to an OLAP (data warehousing) environment. The key elements are:

Shadow/Dual-Run Period: This period allows both systems to operate in parallel, providing a buffer for validation.
Gates Green-Light the Switch: These gates are criteria that must be met before the final switch is made, ensuring minimal risk.

#### Assumptions
The OLTP and OLAP environments have similar data models but might differ slightly due to their operational requirements.
Discrepancies can arise from differences in schema, data handling, or timing of updates.
A robust monitoring system is already in place to track and alert on any issues during the dual-run period.
Step-by-Step Solution
1. Pre-Cutover Planning
Schema Alignment: Ensure that the OLTP and OLAP schemas are aligned as closely as possible, but allow for necessary differences.
Data Synchronization: Implement a mechanism to periodically sync data from the OLTP system to the warehouse.
Monitoring Tools: Deploy monitoring tools to track any discrepancies between the two systems.
2. Dual-Run Period
Synchronous Data Updates: During this period, ensure that all critical transactions in OLTP are replicated to the warehouse as close to real-time as possible.
Data Validation: Regularly compare data from both sources using predefined validation rules and metrics.
Issue Resolution: Address any discrepancies or issues identified during the dual-run phase.
3. Final Cutover Criteria
Data Consistency Check: All critical data should be consistent between OLTP and OLAP.
Performance Metrics: Ensure that the performance of the new system meets the SLAs defined by the business.
User Acceptance Testing (UAT): Conduct UAT to verify the functionality of the dashboard in its final state.
4. Cutover Execution
Shut Down OLTP: Once all gates are green-lighted, shut down the OLTP system and fully transition to the warehouse.
Post-Cutover Monitoring: Continue monitoring post-cutover for any issues that may arise.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary

A well-designed cutover strategy involves a thorough preparation phase, a dual-run period with continuous validation, and stringent criteria to ensure the switch is made only when all systems are in sync. This approach minimizes risk and ensures that critical dashboards continue to provide accurate and timely data without disruptions.

#### Common Mistakes/Follow-Ups
Ignoring Discrepancies: Disregarding small discrepancies during the dual-run period can lead to significant issues post-cutover.
Insufficient Monitoring: Failing to continuously monitor both systems during the dual-run phase can result in undetected issues.
Overlooking Performance Metrics: Neglecting performance metrics might lead to a system that does not meet business requirements.

#### Follow-Ups
What are some potential issues you might encounter during the dual-run period?
How would you handle scenarios where data inconsistencies cannot be resolved before the cutover date?
Can you discuss how you would integrate this strategy with existing backup and recovery procedures?

**Comparison:**

| table_name | columns        |
|------------|----------------|
| column1    | STRING         |
| column2    | INT            |
| sync_status | id  | last_sync_time       |
|-------------|-----|---------------------|
|             | 1   | 2023-10-01T14:30:00Z |

**Code:**

```sql
-- Example SQL for synchronizing data during dual-run period

INSERT INTO olap_database.table_name (column1, column2)
SELECT column1, column2
FROM oltp_database.table_name
WHERE last_update_time > (SELECT last_sync_time FROM sync_status);
```

**Example input / output:**

```text
Input:
- OLTP transaction: INSERT INTO oltp_table (column1, column2) VALUES ('value1', 'value2');
- Sync status: {"last_sync_time": "2023-10-01T14:30:00Z"}

Output:
- OLAP transaction: INSERT INTO olap_table (column1, column2) VALUES ('value1', 'value2');
```

**Data:**

```json
{
  "schema": {
    "table_name": {
      "columns": ["column1", "column2"],
      "last_update_time": "timestamp",
      "last_sync_time": "timestamp"
    }
  },
  "sync_status": [
    {"id": 1, "last_sync_time": "2023-10-01T14:30:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Data Consistent?
5. Proceed with Cutover
6. Resolve Discrepancies
7. End

