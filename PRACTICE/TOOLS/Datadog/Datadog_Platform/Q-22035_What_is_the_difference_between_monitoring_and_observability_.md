---
type: "practice"
difficulty: "Deep"
---

# What is the difference between monitoring and observability and where does Datadog sit?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary: In this answer, we'll explore the nuanced differences between monitoring and observability, with a focus on how tools like Datadog fit into these concepts. We’ll start by defining both terms from first principles before diving deep into their practical implications and where Datadog stands in this landscape.

#### Triage

Monitoring vs Observability: Monitoring is about alerting on known issues, while observability is a broader concept that includes understanding the state of your system and diagnosing problems.
Datadog's Role: Datadog offers tools for both monitoring and observability but emphasizes observability through its comprehensive telemetry stack.

Step-by-Step Solution:

Defining Monitoring

Monitoring, at its core, is about setting up alerts on predefined conditions. For instance, you might configure a rule to alert when the CPU utilization exceeds 90% or when an application crashes. The goal here is proactive issue detection and resolution by keeping an eye on known issues.

Understanding Observability

Observability goes beyond monitoring. It’s about having the ability to understand what's happening within your system at any given time, regardless of whether a predefined rule has been set up. This means you can diagnose problems even when they are not immediately obvious or have not triggered a specific alert. Key aspects include:

Telemetry: Collecting detailed data (metrics, logs, traces) from various sources.
Correlation: Ability to correlate different types of telemetry across services and layers.
Explainability: Being able to explain the state of your system through visualizations and insights.
Datadog in Monitoring

Datadog is a powerful tool that excels at monitoring. It allows you to set up alerts, visualize metrics, and manage logs. The platform provides dashboards where teams can monitor key performance indicators (KPIs) and set thresholds for alerts. For example, you might configure an alert rule that triggers if the response time of your application exceeds 100ms.

Datadog in Observability

Datadog also supports observability by providing a comprehensive suite of tools to gather, correlate, and visualize telemetry data from various sources. Key features include:

Metrics: Collecting detailed metrics from applications, services, and infrastructure.
Logs: Centralizing logs for easy analysis and correlation with other types of telemetry.
Traces: Capturing request traces to understand the flow of requests across different services.
Synthetics: Running automated tests to ensure your application is functioning as expected in production.
Practical Implications

While monitoring helps you catch problems early, observability provides deeper insights into how your system behaves under various conditions. Datadog’s value proposition lies in its ability to seamlessly integrate these capabilities—making it both a robust monitoring platform and an advanced observability tool.

#### Closing Summary
Datadog sits at the intersection of monitoring and observability by offering comprehensive tools for both. While traditional monitoring is about setting up alerts on predefined conditions, observability involves understanding your system's state in real-time. Datadog’s suite of products supports both needs, enabling teams to proactively monitor their systems while also providing deep insights into complex issues.

#### Common Mistakes/Follow-ups

Misunderstanding the Difference: Teams might confuse monitoring and observability as interchangeable concepts.
Overlooking Observability Benefits: Focusing solely on alerting without considering the need for deeper diagnostics.
Ignoring Integration Capabilities: Not leveraging Datadog’s comprehensive suite to fully benefit from both monitoring and observability.

**Comparison:**

```
Metric Type	Monitoring Example	Observability Example
Response Time	Alert when response time > 100ms	Trace to understand request flow and bottlenecks
Logs	Filter logs for errors	Correlate logs with metrics and traces
Traces	None	Visualize end-to-end service performance
```

**Code:**

```python
# Example of setting up an alert in Datadog
def setup_alerts(service):
    dog = datadog.Datadog(api_key)
    check = {
        'init_config': {},
        'instances': [
            {'metric_query': f'avg:{service}.response_time{by(host)} > 100ms', 'name': 'Slow Response'}
        ]
    }
    dog.add_check('slow_response_alert', check)
```

**Example input / output:**

```text
**Input:**
- Monitoring rule: `avg:web_app.response_time > 100ms`
- Observability request: `trace_id:1234567890`

**Output:**
- Alert triggered if the response time exceeds 100ms.
- Trace data for ID 1234567890 showing the entire flow of requests.
```

**Data:**

```yaml
services:
  - name: web_app
    type: http
    url: https://example.com/api/v1/
    timeout: 5s
```

**Flow:**

1. Yes
2. No
3. Monitoring
4. Alert on?
5. Trigger Alert
6. Continue Monitoring
7. Observability
8. Collect Telemetry
9. Correlate Data Across Services
10. Analyze System State inReal-Time

