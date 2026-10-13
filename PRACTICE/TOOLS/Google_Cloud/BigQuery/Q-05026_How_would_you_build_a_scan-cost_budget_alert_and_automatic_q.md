---
type: "practice"
difficulty: "Deep"
---

# How would you build a scan-cost budget alert and automatic query kill for runaway analyst queries using Cloud Monitoring and custom quotas?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ BigQuery Cost Control](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Cost_Control.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of building a scan-cost budget alert and automatic query kill for runaway analyst queries, we need to leverage Google Cloud's Cloud Monitoring along with custom quotas in BigQuery. This solution ensures that analysts are informed when they exceed their allocated budgets or run into high-cost queries that could potentially impact other users.

#### Triage

The key aspects of the question revolve around:

Budget Management: Implementing a budget system to monitor and alert on scan costs.
Automated Query Cancellation: Creating mechanisms to automatically terminate expensive queries before they consume excessive resources or budget.

#### Assumptions
The environment uses Google Cloud BigQuery for data storage and querying.
There is an existing Cloud Monitoring setup in place.
Custom quotas are already configured or can be set up as needed.
Step-by-Step Solution

Setting Up Cost-Based Quotas

Define custom cost-based quotas to limit the scan costs per query, user, or time period.

Creating Alerts with Cloud Monitoring

Utilize Cloud Monitoring to create alert policies that trigger when a query exceeds predefined thresholds.

Automating Query Termination

Use BigQuery APIs and scripts to automatically cancel queries that exceed cost limits.

#### Implementation
Details
1. Setting Up Cost-Based Quotas

To set up cost-based quotas:

Create Custom Quotas
Define Scan Costs Limits
Apply Quotas to Users/Queries

Here’s an example of how you might define a custom quota in the Google Cloud Console or via API:

JSON
{
  "name": "cost_based_quota",
  "metricName": "bigquery.googleapis.com/queries/scanned_bytes",
  "limit": 10485760, // Example: 10 MB (bytes)
  "unit": "BYTES"
}
2. Creating Alerts with Cloud Monitoring

Next, we need to set up alert policies in Cloud Monitoring:

Create Alert Policy
Configure Trigger Conditions
Define Notification Channels

#### Example
configuration for an alert policy:

Metric: bigquery.googleapis.com/queries/scanned_bytes
Threshold: Exceeds 50 MB in 5 minutes
Notification Channel: Slack or email
3. Automating Query Termination

To automate the termination of queries, you can use a combination of BigQuery API and custom scripts:

PYTHON
import google.auth
from google.cloud import bigquery
from datetime import datetime

def terminate_expensive_query(project_id, query_id):
    client = bigquery.Client()
    query_job = client.get_query_job(query_id)

    if (query_job.state != 'DONE' and
        datetime.now() - query_job.start_time > timedelta(minutes=5)):
        query_job.cancel()
        print(f"Query {query_id} terminated due to high cost.")

This script checks the query’s runtime and cancels it if it exceeds a certain threshold.

#### Closing Summary

By combining custom quotas, Cloud Monitoring alerts, and automated query termination scripts, we can effectively manage runaway queries and ensure that costs do not spiral out of control. This approach provides both proactive budget management and real-time response to high-cost queries.

#### Common Mistakes/Follow-Ups
Overlooking Custom Quotas: Failing to set up specific cost-based quotas.
Insufficient Alert Granularity: Setting overly broad alerts can lead to false positives or missed critical thresholds.
Manual Intervention: Relying solely on manual intervention for query termination instead of automated processes.

**Comparison:**

```
Custom Quotas	Description
cost_based_quota	Limits scan costs to 10 MB per query.
Alert Policy Trigger Conditions	Value
Metric Name	bigquery.googleapis.com/queries/scanned_bytes
Threshold	50 MB in 5 minutes
```

**Code:**

```python
import google.auth
from google.cloud import bigquery
from datetime import datetime, timedelta

def terminate_expensive_query(project_id, query_id):
    credentials, _ = google.auth.default()
    client = bigquery.Client(credentials=credentials)
    query_job = client.get_query_job(query_id)

    if (query_job.state != 'DONE' and
        datetime.now() - query_job.start_time > timedelta(minutes=5)):
        query_job.cancel()
        print(f"Query {query_id} terminated due to high cost.")
```

**Example input / output:**

```text
**Input:**
- Set up a custom quota named `cost_based_quota` with a limit of 10 MB per query.
- Configure an alert policy in Cloud Monitoring to trigger when the scanned bytes exceed 50 MB within 5 minutes.

**Output:**
- An automated script that checks for queries exceeding a runtime threshold and terminates them if they are still running.
```

**Data:**

```json
{
  "name": "cost_based_quota",
  "metricName": "bigquery.googleapis.com/queries/scanned_bytes",
  "limit": 10485760,
  "unit": "BYTES"
}
```

**Flow:**

1. Detect Expensive Query
2. Check if Query ExceedsThreshold
3. No, Continue Monitoring

