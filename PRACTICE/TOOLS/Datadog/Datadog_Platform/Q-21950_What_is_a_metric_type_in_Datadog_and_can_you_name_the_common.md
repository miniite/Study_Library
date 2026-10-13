---
type: "practice"
difficulty: "Basic"
---

# What is a metric type in Datadog and can you name the common ones like gauge count and rate?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Datadog, metrics are fundamental data points that represent measurements of system performance. A "metric type" refers to the nature or behavior of how a metric changes over time and can be visualized. Common types include gauge, count, and rate. Let’s dive into each one:

Gauge: This is used for metrics that have a single value at any given point in time, representing a snapshot. For example, the current memory usage on a server.
Count: This type of metric increments or decrements based on events. It's useful for counting occurrences over time. A good example would be the number of requests to an API.
Rate: This represents the rate of change in a count metric per second, helping to measure activity levels more effectively. For instance, if you have 100 requests in one minute, the rate will show how many are coming in per second.

These different types help Datadog users understand and visualize their data in meaningful ways.

**Code:**

```python
# Example code to send metrics with different types using Python SDK (Hypothetical)
from datadog import initialize, api

options = {
    'api_key': 'YOUR_API_KEY',
    'app_key': 'YOUR_APP_KEY'
}
initialize(**options)

# Sending a gauge metric
api.Metric.send(metric='server.memory', points=5000, type="gauge")

# Sending a count metric
def on_request_received():
    api.Metric.increment('api.requests.received')

on_request_received()

# Sending a rate metric
def calculate_rate():
    # Logic to calculate and send the rate of requests per second
    pass

calculate_rate()
```

