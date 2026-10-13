---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you would build a data health status page that non-technical stakeholders can read.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a data health status page that is comprehensible to non-technical stakeholders, we need to focus on simplicity and clarity while ensuring all critical information about data quality and integrity is captured. The core idea is to present key metrics in an easily digestible format, using visualizations that convey the current state of our data pipelines.

The question primarily probes your understanding of:

Key Metrics: What are the most important indicators of a healthy data pipeline?
Visualization Techniques: How can we effectively represent these metrics without overwhelming the user?
Stakeholder Needs: Understanding what non-technical stakeholders need to know and how they will use this information.
Key Assumptions
Stakeholders are typically executives or business owners who need a quick understanding of data health but lack technical background.
Metrics include data availability, freshness, consistency, and completeness.
Technologies: Use standard tools like Grafana for visualization and alerts.
Step-by-Step Solution

Define Key Metrics

Availability: Percentage of data that is available in the pipeline.
Freshness: How recent the latest data is, often measured by latency from source to destination.

#### Cons
istency: Ensuring data conforms to expected formats and relationships.
Completeness: Checking for missing or incomplete records.

Choose Visualizations

Gauge Charts for availability (e.g., green for 95-100%, yellow for 80-94%, red for <80%).
Line Graphs for freshness, showing trends over time.
Heatmaps or Barcharts for consistency checks based on different sources or systems.
Card Layouts for completeness, displaying counts of missing records.

Alerts and Notifications

Set up alerts to notify stakeholders when any metric falls below a critical threshold (e.g., <90% availability).
Integrate these alerts into the status page via pop-ups or emails.

User Interface Design

Ensure the page is clean, uncluttered, and intuitive.
Provide tooltips for non-technical users to understand specific metrics.

Data Source Integration

Collect data from monitoring tools like Prometheus, Datadog, or custom logging.
Use a centralized dashboard tool (Grafana) to aggregate these metrics.

Deployment and Maintenance

Deploy the status page on a stable server or cloud service.
Schedule regular reviews to update thresholds and visualizations based on feedback.

#### Closing Summary

A well-designed data health status page should empower non-technical stakeholders with clear, actionable insights into their data pipelines' performance. By focusing on key metrics, using effective visualizations, and ensuring the interface is user-friendly, we can provide a comprehensive yet accessible view of our data’s quality and reliability.

#### Common Mistakes/Follow-Ups
Overcomplicating Visuals: Avoid adding too many charts or complex graphics that might confuse stakeholders.
Ignoring Stakeholder Feedback: Regularly gather feedback to ensure the page meets their needs.
Lack of Real-Time Data: Ensure metrics are up-to-date and not static, as this can lead to misunderstandings about current conditions.

**Code:**

```python
# Example Python script for fetching data health metrics (pseudo-code)
import requests

def fetch_metrics():
    # Fetch availability from monitoring tool API
    availability = requests.get('http://monitoring_tool/api/availability').json()

    # Fetch freshness from logging system API
    freshness = requests.get('http://logging_system/api/freshness').json()

    return {
        "availability": availability,
        "freshness": freshness
    }
```

**Example input / output:**

```text
Input:
GET /data-health HTTP/1.1

Output:
HTTP/1.1 200 OK
Content-Type: application/json

{
  "status": "Healthy",
  "metrics": [
    {
      "name": "availability",
      "value": 95,
      "threshold": 80,
      "alert_status": "green"
    },
    {
      "name": "freshness",
      "value": 24,
      "last_updated": "2023-10-01T15:00:00Z"
    }
  ]
}
```

**Data:**

```json
{
  "metrics": [
    {"name": "availability", "value": 95, "threshold": 80},
    {"name": "freshness", "value": 24, "last_updated": "2023-10-01T15:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Fetch Metrics
6. Are All Metrics Healthy?
7. Update UI
8. Notify Stakeholders?
9. Send Alert
10. Log Issue
11. Persist Data

