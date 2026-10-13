---
type: "practice"
difficulty: "Basic"
---

# What is retention in Kafka and how would you decide how long to keep a logs topic?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Kafka Retention and Storage](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_Retention_and_Storage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Kafka, retention refers to the duration or size limit that messages can be stored in a Kafka topic before they are automatically deleted. The primary purpose of retention is to balance storage efficiency and operational costs with data availability needs.

#### Triage
Understanding Retention and Its Decisions
What the Question Is Asking

The question seeks an understanding of how long logs should be retained in a Kafka topic, considering both practical and strategic factors like cost, regulatory compliance, and operational requirements.

#### Assumptions
and Context
Storage Costs: Storing data indefinitely can lead to significant costs.
Compliance Requirements: Certain industries have strict regulations about data retention periods (e.g., GDPR for EU users).
Operational Impact: Retention policies affect the load on Kafka brokers, storage efficiency, and backup strategies.

#### First-Principles Solution
Step 1: Define Data Importance

Determine which logs are critical vs. those that can be safely archived or deleted. For example:

Critical Logs: Error messages, system health checks.
Less Critical Logs: User activity logs, non-critical application logs.
Step 2: Consider Regulatory and Compliance Requirements

Check if there are any specific legal or industry standards for data retention times (e.g., GDPR requires a minimum of 3 years for certain types of personal data).

Step 3: Evaluate Operational and Cost Implications
Storage Costs: Calculate the cost implications of retaining logs. Higher storage means higher costs.
Backup Strategy: Determine if regular backups are needed, and if so, how frequently.
Step 4: Define Retention Periods

Based on the above considerations, set retention periods for different types of logs:

Critical Logs: Short term (e.g., 30 days) with frequent backups.
Less Critical Logs: Longer term (e.g., 1 year or more), but still with some form of archival.
Step 5: Implement and Monitor

Implement the retention policy using Kafka's built-in tools, such as log.retention.hours for time-based retention or log.retention.bytes for size-based retention. Regularly monitor storage usage to ensure policies are being adhered to and make adjustments if necessary.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake 1: Retaining all logs indefinitely without considering costs.
Common Mistake 2: Not aligning retention policies with regulatory requirements.

#### Follow-Up
Discuss the balance between keeping data for too long and not having enough historical context.

#### Example Scenario

Suppose we are dealing with an e-commerce application where critical logs (system errors, transactions) need to be retained for up to a year due to compliance reasons. Less critical logs like user activity might only need to be retained for 30 days before being archived or deleted.

**Comparison:**

```
Log Type	Retention Period
System Errors	30 Days
Transaction	1 Year
User Activity	30 Days (Archive)
Non-Critical	7 Days (Delete)
```

**Code:**

```python
# Example configuration snippet in Kafka producer properties
producer_config = {
    "bootstrap.servers": "localhost:9092",
    "acks": "all",
    "compression.type": "gzip",
    "log.retention.hours": 72,  # Retain messages for 3 days before deleting
}
```

**Example input / output:**

```text
Input (Logs Produced):
- Error messages: "Error occurred at 2023-10-01T14:00:00Z"
- User activity logs: "User logged in at 2023-10-01T15:00:00Z"

Output (Retained vs. Deleted):
- Error messages retained until 2023-10-08T14:00:00Z
- User activity logs deleted after 7 days, archived for 3 months before deletion
```

**Data:**

```yaml
# Example Kafka topic configuration
topic_config:
  log.retention.hours: 720  # Retain logs for 1 month before archiving
  cleanup.policy: delete  # Enable deletion of old logs
```

**Flow:**

1. Yes
2. No
3. Monitor Storage Usage
4. Is Storage Cost High?
5. Implement Archival Strategy
6. Adjust Retention Policies
7. Ensure Compliance
8. Continue Monitoring

