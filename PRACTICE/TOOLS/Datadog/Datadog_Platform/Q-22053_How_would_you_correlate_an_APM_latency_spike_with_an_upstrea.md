---
type: "practice"
difficulty: "Deep"
---

# How would you correlate an APM latency spike with an upstream dependency using the Service Map and trace analytics rather than guessing?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ APM and Trace Analytics](<../../../../TOOLS/Datadog/Datadog_Platform/APM_and_Trace_Analytics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To correlate an APM latency spike with an upstream dependency, leveraging Datadog's Service Map and trace analytics is a powerful approach that goes beyond mere guesswork. This involves understanding how requests propagate through your system and identifying bottlenecks or performance issues in dependencies. Let’s break this down from first principles.

TL;DR

First, we’ll use Datadog’s Service Map to visualize the flow of requests between different services. Then, by analyzing traces within the Service Map, we can pinpoint where latencies occur in upstream dependencies and correlate these with specific events or metrics. This method ensures a systematic approach to troubleshooting without relying on guesswork.

#### Triage

The question revolves around using Datadog’s tools—specifically its Service Map and trace analytics—to identify performance issues in the context of an APM latency spike. The key here is not just identifying spikes but understanding their root causes, particularly when they are related to upstream dependencies.

#### Assumptions
You have access to Datadog’s Service Map and Trace Analytics features.
Your system has Datadog agents or integrations in place for monitoring.
You need a systematic approach that can handle complex microservices architectures with multiple tiers of dependencies.
Step-by-Step Solution
1. Visualize the Service Map

The first step is to use Datadog’s Service Map, which provides an overview of how your services interact with each other. Here’s a high-level workflow:

Service A
Service B
Service C

This diagram shows a simplified dependency tree where A depends on B, and B depends on C. In practice, the Service Map would provide more detailed information about request flows, latency, error rates, and so forth.

2. Analyze Traces

Once you have identified potential performance issues from the Service Map, dive into trace analytics to understand where exactly the latencies occur:

Identify Spikes: Look for specific spikes in the service map that coincide with the APM latency spike.

Filter by Dependency: Focus on traces related to the suspected upstream dependencies.

Analyze Trace Details:

Latency Distribution: Use Datadog’s trace filtering and sorting features to find traces where latency is abnormally high.
Error Rate: Check for any correlated error rates or anomalies in specific dependencies.

Correlation with Metrics: Correlate the traces with other metrics such as response times, request volumes, etc., to get a comprehensive view.

3. Use Datadog’s Anomaly Detection

Leverage Datadog’s anomaly detection features to automatically flag suspicious patterns that might indicate performance issues:

APM Latency Spike
Service Map
Trace Analytics
Anomaly Detection

This flow diagram illustrates the process of starting from an observed anomaly in latency, moving through visualizing service interactions, analyzing traces, and finally applying automated detection to identify root causes.

#### Closing Summary

By using Datadog’s Service Map and trace analytics, you can systematically correlate APM latency spikes with upstream dependencies. This approach ensures that troubleshooting is based on actual data rather than guesswork, leading to more effective resolution of issues.

#### Common Mistakes
and Follow-Ups
Mistake 1: Focusing solely on symptoms without understanding the underlying service interactions.
Mistake 2: Not leveraging all available analytics tools within Datadog, such as anomaly detection or detailed trace analysis.

#### Follow-Up
Consider integrating other monitoring tools or services if Datadog alone does not provide enough visibility into your complex microservices architecture.

**Comparison:**

```
Trace ID	Service	Duration (ms)	Span ID
123456	A	900	789012
123456	B	3000	123456
123456	C	1200	654321
```

**Code:**

```python
# Example Python code to filter traces based on high latency
import datadog_api_client

api_instance = datadog_api_client.v1.trace.TraceApi()
api_key = 'your-api-key'
service_name = 'upstream-dependency'

response = api_instance.get_trace_service(service_name,
                                          filters={"status": "error", "latency": "high"})
print(response)
```

**Example input / output:**

```text
Input:
- APM Latency Spike in Datadog
- Service Map highlighting `Service B` as a potential bottleneck

Output:
- Traces showing high latency and error rates for `Service B`
- Metrics indicating increased request volume or other anomalies
```

**Data:**

```json
{
  "traceId": "123456",
  "service": "upstream-dependency",
  "duration": 900,
  "spanID": "789012"
}
```

**Flow:**

1. APM Latency Spike
2. Service Map
3. Trace Analytics
4. Anomaly Detection

