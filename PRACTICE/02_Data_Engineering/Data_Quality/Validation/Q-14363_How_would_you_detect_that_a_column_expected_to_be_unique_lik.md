---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect that a column expected to be unique like a user id has started getting duplicates?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Uniqueness Checks](<../../../../02_Data_Engineering/Data_Quality/Validation/Uniqueness_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect whether a column, such as a user ID, is starting to contain duplicate values, we need to implement a robust data validation process that monitors the uniqueness of this critical field over time. At an intermediate level, the solution involves setting up a periodic or on-demand check for duplicates and leveraging tools like SQL queries and ETL pipelines.

#### Triage

The core question here is how to identify when duplicate user IDs start appearing in our dataset, which could indicate data quality issues such as accidental inserts or upstream system failures. The interviewer might be probing your understanding of database constraints, normalization, and the importance of maintaining unique keys in a database. Assumptions include that we have access to historical data and can run queries on it.

#### First-Principles Solution

Initial Setup:

Ensure that your database has a unique constraint defined on the user ID column. This prevents accidental duplicates but doesn't detect them after they've occurred.

Periodic Check for Duplicates:

Schedule a job to run periodically (e.g., daily) or trigger it manually when suspicious activity is detected.
Use SQL queries to count duplicate values. For example, you can use:
SQL
SELECT COUNT(*) AS duplicate_count, user_id
FROM users
GROUP BY user_id
HAVING COUNT(*) > 1;
Store the results of these checks in a log table for future analysis.

Threshold-Based Alerting:

Define thresholds (e.g., if more than 5 duplicate records are found) to trigger alerts.
Integrate this with monitoring tools like Prometheus or Grafana to send notifications via email, Slack, etc.

Impact Assessment and Remediation:

If duplicates are detected, assess the impact by looking at additional metadata (e.g., timestamps, user attributes).
Depending on severity, decide whether immediate manual intervention is needed or if automated deduplication should be applied.

Continuous Improvement:

Implement feedback loops to improve data quality over time (e.g., better data validation during ingestion).

#### Closing Summary

By setting up a systematic approach involving periodic checks and alerting mechanisms, we can proactively detect and mitigate issues with unique columns like user IDs. This involves a combination of database constraints, SQL queries, logging, and automated alerts to ensure high data integrity.

#### Common Mistakes/Follow-Ups
Mistake: Focusing only on the initial setup without considering ongoing monitoring and alerting.

#### Follow-Up
How would you handle false positives in your system?

Would you like to dive deeper into any part of this solution or explore additional scenarios?

**Comparison:**

| Date       | Duplicate Count | User IDs |
|------------|-----------------|----------|
| 2023-10-01 | 5               | A123456  |
| 2023-10-02 | 0               |          |

**Code:**

```sql
-- Example SQL query for detecting duplicates
SELECT COUNT(*) AS duplicate_count, user_id
FROM users
GROUP BY user_id
HAVING COUNT(*) > 1;
```

**Example input / output:**

```text
Input:
SQL Query to detect duplicates

Output:
+-------------------+
| duplicate_count   | user_id     |
+-------------------+
| 5                 | A123456    |
+-------------------+

Explanation:
The query detected 5 duplicate entries for the user ID 'A123456', indicating a potential issue.
```

**Data:**

```json
{
  "user": {
    "id": "A123456",
    "name": "John Doe"
  },
  "timestamp": "2023-10-01T10:00:00Z"
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is it Duplicate?
5. Log and Alert
6. Continue Monitoring
7. Mitigation Plan
8. Feedback Loop forImprovement

