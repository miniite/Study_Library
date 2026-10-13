---
type: "practice"
difficulty: "Intermediate"
---

# What is StatsD and DogStatsD and how do applications send custom metrics through it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: StatsD and DogStatsD are tools used in Datadog to collect and aggregate application performance metrics. Applications send these metrics over a network using a simple UDP or HTTP protocol, which then get processed by the Datadog backend.

Question Overview: The question is asking about two components of Datadog's system for collecting custom application metrics: StatsD and DogStatsD. The interviewer wants to understand how applications send their own metrics to Datadog using these tools. This involves understanding both the protocols used (UDP or HTTP) and how metrics are structured.

#### Assumptions

We are dealing with a distributed system where various services need to report performance data.
StatsD is an older tool, while DogStatsD is its enhanced version that works better with modern applications.

Step-by-Step Solution:

Understanding StatsD:

StatsD is the original client library for sending metrics to Datadog's backend.
It supports various types of metrics such as counters, gauges, histograms, and sets.
StatsD operates by sending metric data in a simple line protocol over UDP or HTTP.

Metrics Format:

A typical metric sent via StatsD follows the format: <metric_name>:<value>|<type> (e.g., requests:10|c).
requests: The name of the metric.
10: The value of the metric.
c: The type, which can be c for counter, g for gauge, ms for histogram, etc.

Sending Metrics:

Applications use StatsD client libraries to send metrics.

#### Example
in Python using a StatsD client:
PYTHON
from datadog import statsd

# Sending a simple counter metric
statsd.increment('my_counter')
This command sends the my_counter with value 1 (default) and type c.

HTTP Protocol:

For environments where UDP might be problematic (e.g., firewalls), StatsD supports sending metrics over HTTP.

#### Example
using the HTTP protocol:
PYTHON
statsd.increment('my_counter', use_http=True)

DogStatsD:

DogStatsD is an enhanced version of StatsD, designed to work better with modern applications and integrate more seamlessly into Datadog's ecosystem.
It adds support for additional features like tags, service names, metadata, and metrics naming conventions.

#### Example
in Python using DogStatsD:
PYTHON
from datadog import initialize, statsd

options = {
    'api_key': 'YOUR_API_KEY',
    'app_key': 'YOUR_APP_KEY'
}
initialize(**options)

# Sending a gauge metric with tags and service name
statsd.gauge('my_gauge', 50.3, tags=['environment:production'], service='my_service')

Metrics Collection by Datadog Backend:

Once metrics are sent via StatsD or DogStatsD, they are collected by the Datadog backend.
The backend processes and aggregates these metrics for visualization and analysis.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Ensure that applications use the correct metric types (counter, gauge, histogram) based on their needs.
Be aware of potential issues with UDP reliability in environments where packet loss is common.
Use DogStatsD for its enhanced features and better integration with Datadog's platform.

#### Common mistakes
include sending incorrect metric types or not using tags and service names to provide more context. Always refer to the official documentation for specific syntax and usage guidelines.

