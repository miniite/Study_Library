---
type: "practice"
difficulty: "Deep"
---

# A candidate says 'we'll just use SNAPSHOT isolation on the OLTP primary for reports so they see consistent data' — as bar-raiser, what deeper trade-offs do you probe about version-store bloat and long-running reads?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Separating OLTP and Analytics](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the candidate's suggestion of using SNAPSHOT isolation on an OLTP primary database to serve reporting purposes, we need to delve into the deeper trade-offs involved, particularly focusing on version-store bloat and long-running reads.

First, let’s summarize the core issue. The candidate proposes a straightforward solution: leveraging SNAPSHOT isolation on the OLTP primary for generating reports. However, this approach has significant implications that go beyond just making the query results consistent.

#### Triage

The key question here is whether using SNAPSHOT isolation in an OLTP (Online Transaction Processing) environment to serve reporting purposes will effectively balance performance and consistency while mitigating potential downsides such as version-store bloat and the risks associated with long-running reads. The interviewer is probing for a deep understanding of database isolation levels, transaction management, and their impacts on both OLTP and OLAP (Online Analytical Processing) workloads.

#### Assumptions
SNAPSHOT Isolation: This isolation level ensures that transactions see a consistent snapshot of the data at the start of the transaction.
OLTP Workload: Characterized by frequent writes and reads, with ACID properties prioritized for reliability and consistency in real-time transactions.
OLAP Workload: Typically involves complex queries over large datasets, aiming for historical analysis and reporting.

#### First-Principles Solution

Version-Store Bloat:

SNAPSHOT isolation requires maintaining a version store that records the history of all data changes since the transaction started. This can lead to significant storage overhead.
For high-transaction environments (OLTP), this version-store bloat can quickly become problematic, consuming valuable disk space and potentially impacting write performance.

Long-Running Reads:

In an OLTP environment, transactional workloads often have short durations. However, reporting queries under SNAPSHOT isolation might run for much longer periods.
Long-running reads can degrade the performance of both the report generation process and the underlying database, as it locks out other transactions during its execution.

#### Cons
istency vs. Performance:

While SNAPSHOT isolation ensures that reports see a consistent snapshot at transaction start time, it may not be optimal for real-time OLTP operations.
The candidate’s solution doesn’t adequately address potential conflicts between concurrent reporting and transactional workloads.

#### Closing Summary

In summary, while the proposed approach of using SNAPSHOT isolation can provide consistency in report generation from an OLTP primary database, it introduces significant trade-offs. These include version-store bloat, which can impact storage efficiency and write performance, and long-running reads that could degrade overall system throughput. A balanced solution might involve a hybrid approach, such as utilizing a secondary read replica for reporting while maintaining the primary for OLTP operations.

#### Common Mistakes/Follow-Ups
Mistake: Failing to consider the version-store bloat implications.

#### Follow-Up
What steps can be taken to mitigate version-store bloat and ensure efficient use of storage? (e.g., using incremental snapshots, archiving historical data).
Mistake: Ignoring the performance impact of long-running reads on other transactions.

#### Follow-Up
How might we design a solution that minimizes the impact of reporting queries on transactional workloads?

**Comparison:**

| Transaction ID | Isolation Level  | Read Duration (min) |
|----------------|------------------|--------------------|
| T1             | SNAPSHOT         | 60                 |
| T2             | REPEATABLE READ  | 15                 |

**Code:**

```sql
-- Example SQL to illustrate SNAPSHOT isolation usage (hypothetical)
BEGIN TRANSACTION;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
SELECT * FROM sales ORDER BY date_time DESC FOR SYSTEM_TIME AS OF '2023-10-01';
COMMIT;
```

**Example input / output:**

| Transaction ID | Isolation Level | Start Time        | End Time          | Read Duration |
|----------------|----------------|------------------|------------------|---------------|
| T1             | SNAPSHOT       | 2023-10-01 14:00  | 2023-10-01 15:00  | 60 minutes    |
| T2             | REPEATABLE READ| 2023-10-01 14:30  | 2023-10-01 14:45  | 15 minutes    |

**Data:**

```json
{
  "version_store_usage": {
    "oltp_workload": "High",
    "olap_read_duration": "Long",
    "storage_concerns": ["bloat", "write_performance"]
  }
}
```

**Flow:**

1. OLTP Transaction
2. Isolation Level?
3. SNAPSHOT
4. Version-Store Bloat
5. Long-Running Reads
6. Storage Overhead
7. Performance Degradation

