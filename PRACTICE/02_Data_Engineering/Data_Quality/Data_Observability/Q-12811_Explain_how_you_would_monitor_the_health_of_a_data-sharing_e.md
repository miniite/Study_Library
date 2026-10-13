---
type: "practice"
difficulty: "Deep"
---

# Explain how you would monitor the health of a data-sharing/exchange feed you provide to an external partner.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure the health and reliability of a data-sharing/exchange feed provided to an external partner, we need to implement robust data observability practices. This involves monitoring various aspects of the data pipeline, such as data quality, latency, availability, and consistency, to detect issues early and ensure seamless operation.

#### Triage

The question is essentially asking how to set up a comprehensive monitoring system for a data feed that's shared with an external partner. The interviewer is probing your understanding of the various metrics that should be monitored, the tools you would use, and the overall architecture of the solution. Assumptions include:

We are dealing with structured data (e.g., CSV, JSON) or semi-structured data.
The feed has a defined schema.
There are external partners who need timely and accurate data.

#### First-Principles Solution
Data Quality Metrics: Monitor schema adherence, missing values, duplicates, and outliers.
Latency Metrics: Measure the time taken for data to move from ingestion to availability in the feed.
Availability Metrics: Ensure that the data is consistently available and accessible at the expected times.

#### Cons
istency Metrics: Track any discrepancies between source systems and the shared data.
Step-by-Step Solution

Ingestion Layer:

Use tools like Apache Kafka, Amazon Kinesis, or Google Pub/Sub to ingest raw data streams.
Implement validation rules using frameworks like Apache Beam or Spark to ensure schema adherence.

Transformation/Processing Layer:

Apply business logic and transformations on the ingested data.
Use tools like Apache Flink for real-time processing or Apache Airflow for batch processing.
Monitor transformation logs and error rates to detect any issues early.

Storage Layer:

Store processed data in a database (e.g., Amazon Redshift, Google BigQuery) or a data lake (e.g., AWS S3, Azure Data Lake).
Ensure the storage layer is optimized for query performance and scalability.
Monitor disk usage, I/O operations, and query latency.

Export Layer:

Use tools like Apache Nifi or custom scripts to export processed data into a format suitable for external partners (e.g., CSV, JSON).
Schedule exports using CRON jobs or task schedulers.
Ensure the exported files are consistent with expected formats and schemas.

Monitoring Tools:

Utilize tools like Prometheus, Grafana, or AWS CloudWatch to collect and visualize metrics.
Set up alerts for critical thresholds (e.g., latency > 10 seconds, availability down).
Use ELK Stack (Elasticsearch, Logstash, Kibana) for detailed logging and anomaly detection.

Feedback Mechanism:

Provide external partners with feedback mechanisms to report issues or request data updates.
Set up a support ticketing system using tools like JIRA or ServiceNow.

Automated Checks:

Implement automated checks for consistency between source systems and the shared feed.
Use checksums, hashes, or version control metadata to ensure data integrity.

Continuous Improvement:

Regularly review monitoring metrics and system performance.
Optimize processes based on observed trends and feedback from partners.

#### Closing Summary

By implementing a robust observability framework that covers all aspects of the data pipeline—from ingestion to export—along with comprehensive monitoring, automated checks, and continuous improvement, we can ensure that the shared feed is reliable and meets the expectations of external partners. This approach not only ensures timely detection of issues but also enhances overall system resilience.

#### Common Mistakes/Follow-Ups
Ignoring Data Quality: Failing to validate data quality metrics can lead to undetected errors.
Inadequate Latency Monitoring: Poor latency monitoring may result in delayed data availability.
Lack of Consistency Checks: Missing consistency checks between sources and targets can cause discrepancies.
Overlooking Feedback Mechanisms: Without a mechanism for partners to report issues, problems may go unnoticed.

For follow-ups, the interviewer might ask about specific tools used, how you handle large-scale data feeds, or your experience with integrating observability into existing systems. Be prepared to discuss trade-offs between cost and performance and to share relevant technical details.

**Comparison:**

```
Ingestion Layer
Data Validation
Transformation/ProcessingLayer
Storage Layer
Export Layer
Monitoring Tools
Feedback Mechanism
Automated Checks
Continuous Improvement
```

**Code:**

```python
# Example of setting up a simple alert in Prometheus using Grafana
def setup_prometheus_alert():
    template = """
    alert "High Latency"
      if rate(data_source_latency[5m]) > 10ms
      for 30s
      labels {severity="critical"}
      annotations {
        summary = "Data source latency is high",
        description = "Latency exceeded threshold of 10ms over the last 5 minutes."
      }
    """
    return template

# Example script to export data using Apache Nifi
def export_data_to_nifi():
    nifi_url = "http://nifi_host:8080"
    payload = {
        "uri": "/export",
        "data": {
            "file_format": "json",
            "schema": "example_schema",
            "destination": "s3://bucket_name/data_export",
            "timestamp": "2023-10-01T00:00:00Z"
        }
    }
    response = requests.post(f"{nifi_url}/flowFile", json=payload)
    print(response.status_code, response.json())
```

**Example input / output:**

| Timestamp       | Partner ID | Data Quality Issues                         | Latency (seconds) |
|-----------------|------------|--------------------------------------------|------------------|
| 2023-10-01T15:00 | 42         | missing values in column A                  | 8.5              |
| 2023-10-01T16:00 | 42         | No issues                                   | 9.2              |

**Data:**

```json
{
  "schema": {
    "fields": [
      {"name": "timestamp", "type": "string"},
      {"name": "partner_id", "type": "integer"},
      {"name": "data_quality_issues", "type": "array"},
      {"name": "latency_seconds", "type": "number"}
    ]
  },
  "example_data": [
    {
      "timestamp": "2023-10-01T15:00:00Z",
      "partner_id": 42,
      "data_quality_issues": ["missing values in column A"],
      "latency_seconds": 8.5
    },
    {
      "timestamp": "2023-10-01T16:00:00Z",
      "partner_id": 42,
      "data_quality_issues": [],
      "latency_seconds": 9.2
    }
  ]
}
```

**Flow:**

1. Ingestion Layer
2. Data Validation
3. Transformation/ProcessingLayer
4. Storage Layer
5. Export Layer
6. Monitoring Tools
7. Feedback Mechanism
8. Automated Checks
9. Continuous Improvement

