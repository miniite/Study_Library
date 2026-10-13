---
type: "practice"
difficulty: "Intermediate"
---

# What is log indexing in Datadog and how does it relate to retention and cost?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Database Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Database_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Log indexing in Datadog refers to the process of collecting, storing, and processing logs from various sources to enable efficient querying and analysis. This is crucial for monitoring and troubleshooting applications and infrastructure. The relationship between log indexing, retention policies, and cost management is fundamental for effective AIOps practices.

#### Triage

The core of this question revolves around understanding how Datadog handles log data at different stages—from ingestion to long-term storage—and the trade-offs involved in terms of cost versus operational needs. Key assumptions include the familiarity with basic Datadog concepts, such as services and logs, and an understanding that cost optimization is a critical aspect for any AIOps tool.

#### First-Principles Solution
Ingestion: Logs are collected from various sources like servers, applications, or custom integrations.
Indexing & Storage:
Primary Indexes: These store metadata (like timestamps and log levels) that allow for quick querying.
Data Buckets: The actual log content is stored here.
RetentionPolicy: Datadog sets a retention period to determine how long logs should be retained before being deleted. This can range from a few days to years, depending on the service and its logging needs.
Cost Management:
Storage Costs: Longer retention periods lead to higher storage costs.
Query Optimization: Efficient querying strategies need to balance between quick access and cost.

#### Closing Summary

Understanding log indexing in Datadog is essential for managing both operational efficiency and budget constraints. By setting appropriate retention policies, organizations can optimize their logging workflows while keeping costs under control.

#### Common Mistakes/Follow-Ups
Mistake 1: Not considering the trade-offs between long-term storage and query performance.

#### Follow-Up
1: How do different log types (e.g., high-frequency versus low-frequency) impact indexing and retention policies?
Mistake 2: Ignoring the cost implications of excessive data storage.

#### Follow-Up
2: What strategies can be employed to optimize storage costs without compromising on data availability?

**Comparison:**

```
Field	Description
@timestamp	Time at which the log was generated
log_level	Severity level of the log (e.g., info, error)
```

**Code:**

```text
# Example Datadog API call for setting retention policy
curl -X POST "https://api.datadoghq.com/api/v1/logs/config" \
-H "Content-Type: application/json" \
-H "DD-API-KEY: <your-api-key>" \
-H "DD-APPLICATION-KEY: <your-application-key>" \
-d '{
  "service": "<your-service>",
  "log_storage_retention_days": 30
}'
```

**Example input / output:**

```text
Input:
- Service: web_app
- Log storage retention period set to 60 days

Output:
- Logs for the service `web_app` will be retained for 60 days.
- Primary indexes created for fields `@timestamp` and `log_level`.
- Data bucket size set to 1GB per log entry.
```

**Data:**

```json
{
  "service": "web_app",
  "log_storage_retention_days": 60,
  "indexing_settings": {
    "primary_indexed_fields": ["@timestamp", "log_level"],
    "data_bucket_size": "1GB"
  }
}
```

**Flow:**

1. Log Ingestion
2. Primary Indexing
3. Storage in Data Buckets
4. Retention Policy
5. Cost Management

