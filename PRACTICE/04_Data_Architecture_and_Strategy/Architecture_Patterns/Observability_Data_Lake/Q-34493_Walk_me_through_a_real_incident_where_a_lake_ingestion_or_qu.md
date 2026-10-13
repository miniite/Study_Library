---
type: "practice"
difficulty: "Expert"
---

# Walk me through a real incident where a lake ingestion or query problem caused an AIOps blind spot and how you detected and fixed it.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Detecting Ingestion Changes Behind Drift](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Detecting_Ingestion_Changes_Behind_Drift.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, I'll walk you through a specific incident involving our Observability Data Lake (ODL) where we experienced a critical blind spot during an AIOps alert. The root cause turned out to be a query issue that masked anomalies in the data lake ingestion process. Let's break down how we identified and resolved it.

To start, we need to understand what "Observability Data Lake" means: It is a centralized repository designed for storing and processing various types of telemetry data from our distributed systems. The key challenge here is ensuring consistent and reliable data ingestion and querying capabilities to maintain observability at scale.

During the incident, we received alerts indicating that certain services were experiencing high latency issues, but the anomalies in the metrics did not reflect any corresponding changes in other relevant KPIs. This led us to suspect a potential issue with how the queries were being executed or data was ingested into our Data Lake.

#### Triage
and Assumptions
Assumption 1: The query logic is correct, but there might be an ingestion issue.
Assumption 2: There's a possibility of data corruption or delay in processing that affects the anomaly detection systems.
Assumption 3: The alerting system itself may not have been designed to handle certain types of anomalies effectively.
Step-by-Step Solution

Initial Investigation:

Check Alerts and Metrics: First, we reviewed all relevant alerts and metrics from our monitoring tools (Prometheus, Grafana) to understand the pattern and scope of the issue.
Review Logs: We examined the logs for any error messages or warnings that could indicate issues with the data ingestion pipeline.

Data Validation:

Dry Run Data Ingestion: We performed a dry run of the data ingestion process using known good and bad datasets to see if there were any discrepancies.
Compare Ingested vs Raw Data: We compared the raw telemetry data against what was being ingested into the Data Lake. This helped identify any missing or corrupted data points.

Query Optimization:

Analyze Query Execution Plans: We used tools like PostgreSQL’s EXPLAIN command to analyze query execution plans and look for inefficient queries that might be causing delays.
Check Indexes and Partitions: Ensured that the correct indexes and partitions were in place to optimize query performance.

Root Cause Identification:

Identify Correlated Metrics: We cross-referenced the metrics with other relevant data sources (e.g., system logs, network traces) to find any correlations or patterns.
Reproduce Issue: We attempted to reproduce the issue in a test environment to validate our findings.

Fix and Validate:

Optimize Ingestion Pipeline: We fine-tuned the ingestion pipeline by adding more robust error handling and logging mechanisms.
Improve Query Execution: Optimized queries and ensured they were executed efficiently without delays.

#### Closing Summary

By systematically investigating the alert, validating data, analyzing query execution plans, and improving both the ingestion pipeline and query performance, we identified a critical blind spot caused by a combination of data corruption during ingestion and inefficient querying. This incident highlighted the importance of robust observability practices, including thorough validation processes and continuous monitoring.

#### Common Mistakes/Follow-Ups
Failing to Validate Data Thoroughly: Often, teams focus too much on query optimization without validating whether the data being queried is accurate.
Ignoring Subtle Ingestion Issues: Minor errors in data ingestion can lead to significant blind spots if not properly detected and addressed.

To avoid such issues in the future, we implemented more rigorous validation checks during both development and production stages. We also enhanced our alerting systems to better handle edge cases and anomalies.

**Comparison:**

```
Ingested Data	Raw Data
timestamp: 2023-10-01T12:00:00Z, service_id: A1234567890, metric_value: 2.5	timestamp: 2023-10-01T12:00:00Z, service_id: A1234567890, metric_value: 2.5
timestamp: 2023-10-01T12:01:00Z, service_id: A1234567890, metric_value: 3.0	timestamp: 2023-10-01T12:01:00Z, service_id: A1234567890, metric_value: 3.5
```

**Code:**

```python
# Example of a Python function to validate data ingestion
def validate_data(ingested_data, raw_data):
    for record in ingested_data:
        if not verify_record(record, raw_data):
            print(f"Data validation failed for {record}")
```

**Example input / output:**

```text
Input:
validate_data(ingested_data, raw_data)

Output:
Data validation failed for {'timestamp': '2023-10-01T12:01:00Z', 'service_id': 'A1234567890', 'metric_value': 3.0}
```

**Data:**

```json
{
  "ingested_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "service_id": "A1234567890", "metric_value": 2.5},
    {"timestamp": "2023-10-01T12:01:00Z", "service_id": "A1234567890", "metric_value": 3.0}
  ],
  "raw_data": [
    {"timestamp": "2023-10-01T12:00:00Z", "service_id": "A1234567890", "metric_value": 2.5},
    {"timestamp": "2023-10-01T12:01:00Z", "service_id": "A1234567890", "metric_value": 3.5}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Validate Ingestion
5. Is Data Valid?
6. Log and Continue
7. Fix Ingestion Issue
8. Revalidate
9. End

