---
type: "practice"
difficulty: "Intermediate"
---

# When migrating from an OLTP-only stack to adding a warehouse, how do you decide what stays as operational reporting on the OLTP side vs what moves to OLAP?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ OLTP vs OLAP Workloads](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/OLTP_vs_OLAP_Workloads.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate from an OLTP-only stack to one that includes both operational reporting and analytics (OLAP), we need to carefully evaluate which parts of our data processing should remain in the transactional database and which can be moved to a more analytical environment. The goal is to optimize performance, reduce costs, and ensure data integrity while maintaining functionality.

The key distinction between OLTP and OLAP lies in their primary goals: OLTP databases are designed for high-speed transactions, whereas OLAP databases focus on analyzing large amounts of historical data. Therefore, we need to determine what type of queries and operations are best suited for each environment.

#### Triage

Question: When migrating from an OLTP-only stack to adding a warehouse, how do you decide what stays as operational reporting on the OLTP side vs what moves to OLAP?

What is being probed?: The interviewer is probing your understanding of the differences between OLTP and OLAP, the trade-offs involved in moving data, and your ability to make informed decisions about workload distribution.

#### Assumptions
We have a clear distinction between transactional operations (OLTP) and analytical operations (OLAP).
The goal is to optimize performance and minimize costs by leveraging different types of databases for their respective strengths.

#### First Principles Solution
Step 1: Understand the Workload
Transaction Frequency: Operational reporting queries are often frequent but low in complexity. Analytical queries, on the other hand, can be less frequent but more complex.
Data Volume and Granularity: OLTP systems typically handle high transaction volumes with fine-grained data, whereas OLAP deals with larger volumes of historical or aggregated data.
Step 2: Evaluate Query Characteristics

OLTP Queries:

High frequency (e.g., daily or hourly).
Low complexity (e.g., simple SELECT statements to retrieve current status).
Real-time requirement for accurate data.

OLAP Queries:

Lower frequency (e.g., monthly or weekly).
Higher complexity (e.g., complex JOINs, aggregations over large datasets).
Less real-time requirements but need historical data.
Step 3: Performance and Cost Considerations
Performance: OLTP systems are optimized for fast read/write operations, while OLAP systems can handle more extensive processing tasks.
Cost: Moving data to an OLAP system might introduce additional storage costs. However, the cost of maintaining a transactional database for complex reporting queries could be higher in terms of computational resources.
Step 4: Data Integrity and Consistency
Transaction Integrity: Ensuring that data remains consistent and up-to-date across both systems.
Data Freshness: OLAP systems may use late-binding or snapshotting techniques to maintain a copy of the operational database at specific points in time.

#### Solution

Identify Operational Reporting Queries:

Select queries that are high-frequency, low-complexity, and real-time critical for OLTP.

Identify Analytical Queries:

Isolate complex, less frequent, or historical reporting queries to move to the OLAP system.

Design Data Flow:

Implement a robust ETL (Extract, Transform, Load) process that synchronizes data between the OLTP and OLAP systems.
Ensure real-time updates are captured in the OLTP system while historical snapshots are stored for analysis.

Optimize Performance:

Use appropriate indexing strategies to improve query performance on the OLTP side.
Optimize storage and processing techniques on the OLAP side, such as materialized views or pre-aggregated data.

Monitor and Tune:

Continuously monitor system performance and adjust configurations based on real-world usage patterns.

#### Closing Summary

By carefully evaluating the characteristics of your workload and understanding the strengths of both OLTP and OLAP systems, you can make informed decisions about what should remain in the transactional database and what can be moved to an analytical environment. This approach ensures optimal performance and cost-efficiency while maintaining data integrity and accuracy.

#### Common Mistakes/Follow-Ups
Failing to Recognize Real-Time Requirements: Overlooking queries that require immediate results.
Ignoring Data Freshness: Not considering the need for fresh, up-to-date data in analytical processes.
Not Implementing Robust ETL Processes: Failing to establish a reliable way to sync data between systems.

**Comparison:**

| ID | TransactionTime | Amount | CustomerID | AggregatedField | |----|------------------|--------|------------| | 1 | 2023-10-01T10:00:00Z | 50.00 | 1 | 60 | | 2 | 2023-10-02T14:30:00Z | 75.00 | 2 | 85 |

**Code:**

```sql
-- Example ETL Job (Pseudo-code)
BEGIN TRANSACTION;

-- Extract Data from OLTP
INSERT INTO HistoricalData SELECT * FROM CurrentData WHERE LastUpdated > '2023-10-01';

-- Transform Data as Needed
UPDATE HistoricalData SET AggregatedField = SumOfAnotherField;

-- Load Data into OLAP Store
INSERT INTO OLAPStore SELECT * FROM HistoricalData;

COMMIT TRANSACTION;
```

**Example input / output:**

```text
Input:
- Current Data from OLTP

Transformation Steps:
1. Extract new data entries since the last update.
2. Aggregate and transform as needed.

Output:
- Historical Data stored in a snapshot table for OLAP queries
- Aggregated Data loaded into the OLAP store
```

**Data:**

```json
{
  "CurrentData": {
    "fields": ["ID", "TransactionTime", "Amount", "CustomerID"],
    "rows": [
      {"ID": 1, "TransactionTime": "2023-10-01T10:00:00Z", "Amount": 50.00, "CustomerID": 1},
      {"ID": 2, "TransactionTime": "2023-10-02T14:30:00Z", "Amount": 75.00, "CustomerID": 2}
    ]
  },
  "HistoricalData": {
    "fields": ["ID", "TransactionTime", "Amount", "CustomerID", "AggregatedField"],
    "rows": [
      {"ID": 1, "TransactionTime": "2023-10-01T10:00:00Z", "Amount": 50.00, "CustomerID": 1, "AggregatedField": 60},
      {"ID": 2, "TransactionTime": "2023-10-02T14:30:00Z", "Amount": 75.00, "CustomerID": 2, "AggregatedField": 85}
    ]
  },
  "OLAPStore": {
    "fields": ["CustomerID", "TotalAmount"],
    "rows": [
      {"CustomerID": 1, "TotalAmount": 60},
      {"CustomerID": 2, "TotalAmount": 85}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Extract Data
4. Filter New Entries?
5. Transform Data
6. Load Historical Data
7. Load Aggregated Data
8. Store Historical Data
9. Store Aggregated Data inOLAP Store

