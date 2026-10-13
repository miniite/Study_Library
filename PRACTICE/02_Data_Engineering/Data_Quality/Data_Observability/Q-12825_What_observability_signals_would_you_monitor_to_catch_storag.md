---
type: "practice"
difficulty: "Deep"
---

# What observability signals would you monitor to catch storage/cost anomalies (e.g. a job writing 10x normal data)?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of monitoring storage and cost anomalies, particularly those caused by unexpected increases in data volume, we need to establish a robust observability framework that captures key signals at various stages of the data pipeline. The primary goal is to detect when a job writes significantly more than its usual amount of data, which could indicate issues like data duplication, processing errors, or misconfigured jobs.

The question can be broken down into several key components: identifying the types of observability signals required, understanding the context in which these signals operate, and how they are aggregated to provide meaningful insights. Let's dive deeper using a first-principles approach.

**Comparison:**

```
Metric	Description
current_usage	Latest storage usage for a specific job.
historical_usage	List of past storage usages over time.
```

**Code:**

```python
# Example code for querying metrics from a cloud provider API
import requests

def get_storage_usage(job_id):
    response = requests.get(f"https://api.cloudprovider.com/v1/jobs/{job_id}/metrics")
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception("Failed to fetch metrics")

# Example function for comparing current data usage with historical trends
def compare_current_with_trends(storage_metrics):
    # Fetch recent and historical storage usage data
    recent_usage = storage_metrics['current_usage']
    past_usage = storage_metrics['historical_usage']

    # Calculate anomaly threshold (e.g., 10x the average)
    average_past_usage = sum(past_usage) / len(past_usage)
    anomaly_threshold = 10 * average_past_usage

    return recent_usage > anomaly_threshold
```

**Example input / output:**

```text
Input:
- Job ID: 12345

Output:
- Storage Metrics JSON object from cloud provider API call.
- Comparison result indicating if the job's current usage exceeds an anomaly threshold (True/False).
```

**Data:**

```json
{
  "current_usage": 50GB,
  "historical_usage": [30GB, 40GB, 25GB]
}
```

**Flow:**

1. No
2. Yes
3. Monitor Job Start Time
4. B
5. Metric Collection (e.g., StorageUsage)
6. Is Current Usage > AnomalyThreshold?
7. Log for Investigation
8. Alert Stakeholders

