---
type: "practice"
difficulty: "Deep"
---

# What signals would you use in Datadog to detect a retry storm amplifying load during a partial outage?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Anomaly and Forecast Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Anomaly_and_Forecast_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect a retry storm amplifying load due to a partial outage in Datadog, we need to identify metrics that indicate an increase in retries and correlate them with service availability or error rates. A retry storm happens when a system retries failing requests at an excessive rate, which can amplify the impact of a partial outage by overwhelming other parts of the system.

#### Triage
What Are We Probing?

The key aspect here is understanding how to set up Datadog monitoring for robust service-level objectives (SLOs) and alerting. The question requires us to identify specific metrics like retry counts, error rates, and latency spikes that can indicate a retry storm. Additionally, we need to consider the correlation between these metrics and service availability to pinpoint when a partial outage is causing retries.

#### Assumptions
Service Availability: We assume there's an existing monitoring setup for basic service availability.
Error Rate Metrics: We assume error rates are being tracked in Datadog.
Retry Count Metrics: Retries are counted and monitored as part of the application or infrastructure logs.

#### Solution
From First Principles

Identify Retry Metrics:

Datadog Logs Monitoring: Collect retry logs from applications and services to track retry counts over time.
Custom Datadog Metrics: Create custom metrics for retries per minute, hour, etc., based on log entries.

Error Rate Metrics:

Use Datadog's built-in APM (Application Performance Monitoring) to track error rates.
Monitor HTTP response codes (e.g., 50x) as part of the application logs.

Correlate Metrics with Outage Signals:

Set up alerting on both retry count and error rate metrics for service availability monitoring.
Use Datadog's correlation features to link these metrics when a partial outage is detected.

Visualize Metrics:

Create dashboards in Datadog that aggregate the above metrics over time.
Implement anomaly detection rules to automatically flag unusual increases in retry counts or error rates.

#### Example
Dashboard Setup

Retry Count per Minute:

Yes
Service Name
Retry Count per Minute
Threshold Exceeded?
Alert Generated

Error Rate Monitoring:

Yes
Service Name
HTTP Error Rate (50x)
Threshold Exceeded?
Alert Generated

Correlation Analysis:

Use Datadog's correlation analysis to identify patterns where a rise in retry counts coincides with an increase in error rates.
This helps pinpoint the cause of the partial outage and potential retry storms.

#### Closing Summary

In summary, setting up effective monitoring for retry storms involves tracking retry metrics, error rates, and using Datadog’s advanced correlation features. By visualizing these metrics through custom dashboards and implementing anomaly detection, we can proactively alert on potential issues before they escalate into significant outages.

#### Common Mistakes/Follow-Ups
Ignoring Log Data: Relying solely on Datadog metrics without analyzing application logs can miss critical insights.
Lack of Correlation: Failing to correlate retry counts with error rates and service availability can result in delayed detection of issues.
Threshold Setting: Poorly set threshold values for alerts can lead to either false positives or false negatives.

**Comparison:**

```
Metric	Unit	Description
Retry Count Per Minute	count/minute	Number of retries per minute
Error Rate 50x	percentage	Percentage of HTTP 50x errors
```

**Code:**

```python
# Example Python script for logging retries (simplified)
import logging

logging.basicConfig(level=logging.INFO)

def handle_request(request):
    try:
        # Simulate processing the request
        pass
    except Exception as e:
        logging.warning(f"Failed to process request {request.id}, retrying...")
        raise
```

**Example input / output:**

```text
Input (Sample Logs):
2023-10-01T14:00:00Z - WARN: Failed to process request 123, retrying...
2023-10-01T14:01:00Z - WARN: Failed to process request 456, retrying...

Output (Alert):
Service 'example-service' experienced a spike in retry count from 0 to 10 requests per minute, triggering an alert.
```

**Data:**

```json
{
  "metrics": {
    "retry_count_per_minute": {
      "unit": "count/minute",
      "service_name": "example-service"
    },
    "error_rate_50x": {
      "unit": "percentage",
      "service_name": "example-service"
    }
  },
  "thresholds": {
    "retry_count_threshold": 10,
    "error_rate_threshold": 1
  }
}
```

**Flow:**

1. Yes
2. No
3. Service Request
4. Process Request
5. Request Successful?
6. End Processing
7. Log Error, Raise Exception
8. Retry Handling
9. Monitor Retry Count
10. Correlate with Error Rate
11. Generate Alert if ThresholdExceeded

