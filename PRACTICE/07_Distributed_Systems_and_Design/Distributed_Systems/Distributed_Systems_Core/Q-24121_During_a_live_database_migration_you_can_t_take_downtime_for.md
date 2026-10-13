---
type: "practice"
difficulty: "Expert"
---

# During a live database migration you can't take downtime for a full row-by-row diff. Design a continuous shadow-comparison/reconciliation using sampling and checksums, and describe how you triage the stream of legitimate-vs-bug discrepancies.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Verifying Live Migrations](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Verifying_Live_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a continuous shadow-comparison/reconciliation system during live database migrations, the key is to ensure data consistency without disrupting service. The solution involves using sampling and checksums to continuously compare the source and target databases. This approach allows us to identify legitimate discrepancies and isolate potential bugs or issues that may arise.

#### Triage
What's Being Probed

The interviewer is probing your understanding of:

Real-time Data Comparison Techniques: How to efficiently compare data in real-time without downtime.
Sampling Strategies: Choosing the right sampling techniques to balance accuracy with performance.
Checksum Algorithms: Utilizing checksums for quick verification of data integrity.
Error Handling and Triage: Managing legitimate vs. bug discrepancies effectively.

#### Assumptions
The databases are large, containing billions of rows.
The migration process must be non-disruptive to users.
There is a high volume of changes occurring simultaneously in the database.
The system needs to handle edge cases such as partial updates and deletes.

#### Solution
From First Principles
Step 1: Sampling Strategy

To minimize the computational overhead, we employ sampling techniques. Given that full row-by-row comparisons are impractical due to downtime constraints, sampling allows us to check a representative subset of data points.

Sampling Technique: Use random sampling with stratified sampling if necessary. This ensures that each part of the database is represented.
Sample Size Calculation: Calculate the sample size based on the total number of rows in the database and the acceptable error margin. For example, 1% to 5% of the data might be sampled.
Step 2: Checksum Computation

Checksums provide a quick way to verify data integrity by comparing hash values of the source and target databases. This reduces the need for full comparisons.

Checksum Algorithm: Use a robust checksum algorithm like SHA-256.
Real-time Calculation: Compute the checksum on both the source and target samples in real time. Store these values in a separate metadata table or stream processing system.
Step 3: Real-Time Comparison

Implement a continuous comparison mechanism that:

Polls Data Samples: Periodically fetches sampled data from the source and target databases.
Computes Checksum Differences: Compares the checksums of sampled data to detect discrepancies.
Tracks Discrepancies: Stores identified discrepancies in a structured format for further analysis.
Step 4: Triage Mechanism

Develop a triage system that classifies discrepancies into legitimate and potential bugs:

Legitimate Discrepancies: Common issues like partial updates, concurrent transactions, or network latency can cause discrepancies.
Potential Bugs: Uncommon patterns indicating coding errors in the migration process.
Log Analysis: Log all discrepancies with metadata such as timestamps, affected rows, and involved operations.
Error Patterns Identification: Use statistical analysis to identify patterns that could indicate bugs (e.g., frequent occurrences of certain types of discrepancies).
Step 5: Reporting and Alerting

Set up a reporting system that:

Generates Alerts: Sends notifications when potential bug discrepancies are detected.
Summarizes Findings: Provides regular reports summarizing the state of the migration, including identified issues and their nature.

#### Closing Summary

By leveraging sampling and checksums, we can achieve continuous data reconciliation during live migrations with minimal downtime. The triage system helps in distinguishing between legitimate discrepancies and potential bugs, ensuring that only critical issues are escalated for immediate attention.

#### Common Mistakes/Follow-Ups
Ignoring Edge Cases: Ensure that edge cases such as partial updates and deletes are handled correctly.
Overlooking Performance: Avoid sampling sizes that introduce significant overhead or reduce the effectiveness of the system.
Lack of Logging: Proper logging is crucial for diagnosing issues later on, especially in complex environments.
Triaging Discrepancies

Common Legitimate Issues:

Partial Updates: A row might be partially updated but not fully written to both databases.
Concurrent Transactions: Simultaneous transactions can lead to discrepancies that resolve themselves over time.

Potential Bugs:

Inconsistent Code: Migrating code that introduces bugs, such as incorrect transformations or missing logic.
Network Issues: Persistent network errors causing data corruption.

#### Example
Workflow
PLAINTEXT
1. Sample rows from source and target databases (e.g., 5%).
2. Compute checksums for sampled rows.
3. Compare checksums to detect discrepancies.
4. Log discrepancies with relevant metadata.
5. Analyze logs for patterns indicating potential bugs.
6. Report findings and escalate critical issues.

**Comparison:**

| Row ID | Source Data | Target Data | Discrepancy |
|--------|-------------|-------------|-------------|
| 1      | value1      | value2      | True        |
| 2      | value3      | value4      | False       |

**Code:**

```python
import hashlib

def compute_checksum(data):
    return hashlib.sha256(data.encode()).hexdigest()

# Sample rows from the database
source_rows = fetch_sample_rows(source_db, sample_size)
target_rows = fetch_sample_rows(target_db, sample_size)

for row in source_rows:
    src_checksum = compute_checksum(row)
    tgt_checksum = compute_checksum(fetch_row_from_target_db(row))

    if src_checksum != tgt_checksum:
        log_discrepancy(row, src_checksum, tgt_checksum)
```

**Example input / output:**

```text
Input:
- Source and target databases with billions of rows.
Output:
- Metadata table/log entries containing identified discrepancies.
```

**Data:**

```json
{
  "source_db": {
    "rows": [
      { "id": 1, "data": "value1" },
      { "id": 2, "data": "value2" },
      ...
    ]
  },
  "target_db": {
    "rows": [
      { "id": 1, "data": "value3" }, // Discrepancy
      { "id": 2, "data": "value4" }
    ]
  }
}
```

**Flow:**

1. Discrepancies Detected
2. Potential Bug Identified
3. Legitimate Issue
4. Fetch Sample Rows
5. Compute Checksums
6. Compare Checksums
7. Triage Legitimate vs. Bugs
8. Alert and Escalate
9. Log and Monitor

