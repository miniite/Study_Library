---
type: "practice"
difficulty: "Intermediate"
---

# How do you validate an OLTP-to-OLAP migration end-to-end — row counts, checksums, business-metric reconciliation — before cutting over reporting?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a successful migration from Online Transaction Processing (OLTP) to Online Analytical Processing (OLAP), you must validate several key aspects: row counts, checksums, and business-metric reconciliation. This validation process is crucial before cutting over reporting to avoid data discrepancies or loss. Let’s break down the steps from first principles.

Firstly, the question probes your understanding of the differences between OLTP and OLAP, as well as how they interact during a migration. You need to ensure that the transactional database (OLTP) data is accurately transferred to the analytical database (OLAP), maintaining integrity through various validation checks.

#### Triaging the Question
Row Counts: Verify that the number of rows in the OLTP and OLAP databases match.
Checksums: Ensure that the integrity of the data is maintained during migration by comparing checksums.
Business-Metric Reconciliation: Validate that key business metrics calculated from the OLTP data are consistent with those derived from the OLAP data.

#### Assumptions
The OLTP and OLAP systems have equivalent schemas (or compatible mappings).
There are tools available for performing bulk data migration, checksum generation, and metric calculation.
Both databases support advanced querying and data manipulation operations necessary for validation.
Step-by-Step Solution

Pre-Migration Preparation:

Ensure both OLTP and OLAP systems are in a known good state.
Backup the current production environment to ensure rollback options if needed.

Data Extraction from OLTP:

Extract the relevant data using SQL queries optimized for large datasets.
Use appropriate indexes or partitioning strategies to enhance performance.

Checksum Generation and Validation:

Generate checksums of critical tables in both systems.
Compare these checksums to ensure the integrity of the extracted data during migration.

Data Migration:

Perform a controlled bulk load into the OLAP system.
Monitor for any errors or warnings during the migration process.

Row Count Verification:

Compare row counts between corresponding tables in OLTP and OLAP systems.
Use SQL queries to generate these comparisons, e.g., SELECT COUNT(*) FROM oltp_table WHERE ... vs. SELECT COUNT(*) FROM olap_table WHERE ....

Business-Metric Reconciliation:

Calculate key business metrics from both the OLTP and OLAP data sets.
Compare results for consistency. For example, if sales figures are calculated differently, they should match within a certain threshold.

Post-Validation Checks:

Perform additional checks such as integrity constraints, referential integrity, and any custom validation rules defined in the OLTP system.
Ensure that no data has been lost or corrupted during migration.

#### Closing Summary

The end-to-end validation process involves verifying row counts, checksums, and business-metric reconciliation to ensure a seamless transition from OLTP to OLAP. This thorough approach minimizes risks of data discrepancies and ensures the reliability of the new reporting system.

#### Common Mistakes/Follow-Ups
Mismanaging Data Integrity: Failing to validate checksums can lead to undetected corruption.
Ignoring Business Metrics: Overlooking business metrics can result in incorrect reporting and misinformed decision-making.
Insufficient Monitoring: Not monitoring the migration process for errors or warnings can delay discovery of issues.

**Comparison:**

```
Table	OLTP Row Count	OLAP Row Count
table1	2	2
table2	2	2
```

**Code:**

```sql
-- Example SQL queries for generating checksums
SELECT md5(CAST(STRING_AGG(column1, ','), 'text')) AS checksum FROM oltp_table WHERE condition;
SELECT md5(CAST(STRING_AGG(column1, ','), 'text')) AS checksum FROM olap_table WHERE condition;

-- Example comparison of row counts
SELECT COUNT(*) FROM oltp_table WHERE condition = (SELECT COUNT(*) FROM olap_table WHERE condition);
```

**Example input / output:**

```text
Pre-Migration:
- OLTP row counts: table1 = 2, table2 = 2
- OLAP row counts: table1 = 2, table2 = 2

Post-Migration:
- OLTP checksums: table1 = abcdef01, table2 = bcdef012
- OLAP checksums: table1 = abcdef01, table2 = bcdef012

Business Metrics:
- OLTP total sales: $500
- OLAP total sales: $500 (within 0.1%)
```

**Data:**

```json
{
  "oltpSchema": {
    "table1": [
      {"id": 1, "value": 100},
      {"id": 2, "value": 200}
    ],
    "table2": [
      {"id": 3, "value": 300},
      {"id": 4, "value": 400}
    ]
  },
  "olapSchema": {
    "table1": [
      {"id": 1, "value": 100},
      {"id": 2, "value": 200}
    ],
    "table2": [
      {"id": 3, "value": 300},
      {"id": 4, "value": 400}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Extract Data from OLTP
7. Generate Checksums in BothDatabases
8. Validate Row Counts
9. Calculate Business Metrics
10. Identify and Fix Issues
11. Perform Data Migration
12. Migrate Data to OLAP
13. Monitor for Errors/Warnings
14. All Tests Pass?
15. Complete Migration
16. Fix Issues, Repeat
17. End

