---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between data quality checks and data observability. If you could only instrument three signals on your gold tables first, which and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distinguishing Drift from Pipeline Breaks](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Distinguishing_Drift_from_Pipeline_Breaks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a robust data architecture, both data quality checks and data observability are critical components. Data quality checks ensure that the data is accurate, complete, consistent, and free from errors at ingestion time or in near real-time. On the other hand, data observability provides visibility into how the data flows through the system, enabling real-time monitoring of performance, health, and usage patterns.

To understand this difference more deeply, let's start by defining each concept:

Data Quality Checks: These are proactive measures aimed at ensuring that data meets specific quality standards before it is stored in the warehouse or consumed by downstream systems.
Data Observability: This involves collecting detailed metadata about how and when data flows through various stages of the pipeline, including ingestion, processing, storage, and consumption.

For a Data Engineer with an Architect/Principal role, ensuring that you can make informed decisions based on real-time insights is crucial. Therefore, if I could only instrument three signals first, I would prioritize:

Error Rates: Tracking how often data ingestion fails or produces errors.
Latency: Monitoring the time taken for data to be ingested and processed.
Usage Patterns: Analyzing which parts of the data are frequently accessed and how they change over time.

These signals will provide a comprehensive view into the system's health, performance, and utility, allowing us to identify bottlenecks, optimize processes, and maintain high-quality data.

**Comparison:**

| Key          | Value    |
|--------------|----------|
| Error Rate   | 2.3%     |
| Latency      | 300 seconds |
| Top Usage   | customer_id_1, transaction_id_2, product_id_3 |

**Code:**

```python
# Example code for monitoring error rates
def monitor_error_rates(data):
    errors = 0
    total_records = len(data)
    for record in data:
        if not is_valid_record(record):  # Assume a function that checks validity
            errors += 1
    return (errors / total_records) * 100

# Example code for monitoring latency
import time
start_time = time.time()
ingest_data()  # Assume an ingestion process
end_time = time.time()
latency = end_time - start_time
print(f"Latency: {latency} seconds")

# Example code for analyzing usage patterns
from collections import defaultdict

usage_patterns = defaultdict(int)
for record in data:
    key = get_key(record)  # Assume a function that extracts keys
    usage_patterns[key] += 1

top_usage_keys = sorted(usage_patterns.items(), key=lambda x: x[1], reverse=True)[:5]
print("Top Usage Patterns:", top_usage_keys)
```

**Example input / output:**

```text
Input:
data = [
  {"customer_id": "123", "transaction_time": "2023-09-24T10:00:00Z"},
  {"customer_id": "456", "transaction_time": "2023-09-24T10:01:00Z"},
  # ... (1000 records)
]

Output:
Error Rate: 2.3%
Latency: 300 seconds
Top Usage Patterns:
customer_id_1: 120 times
transaction_id_2: 80 times
product_id_3: 50 times
```

**Data:**

```json
{
  "error_rates": {
    "table_name": "gold_table",
    "errors": 23,
    "total_records": 1000,
    "percentage": 2.3
  },
  "latency": {
    "ingestion_time": "2023-09-25T14:00:00Z",
    "end_time": "2023-09-25T14:05:00Z",
    "total_seconds": 300
  },
  "usage_patterns": [
    {"key": "customer_id_1", "count": 120},
    {"key": "transaction_id_2", "count": 80},
    {"key": "product_id_3", "count": 50}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Ingestion
6. Is Data Valid?
7. Record Ingested
8. Error Recorded
9. Process Next Record
10. Retry or Mark for Deletion
11. Fetched Data
12. Is Data Processed?
13. Store in Warehouse
14. Back to A

