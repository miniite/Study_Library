---
type: "practice"
difficulty: "Intermediate"
---

# What is p95 or p99 latency and how would you display it on a Datadog dashboard?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Composite Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Composite_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand p95 or p99 latency, we first need to clarify what these metrics represent in the context of performance monitoring. Essentially, p95 (percentile 95) and p99 (percentile 99) are statistical measures that tell us about the distribution of response times for a given service or endpoint over time. Specifically:

p95 latency: This means that 95% of all requests to a system have response times less than this value.
p99 latency: Similarly, 99% of all requests have response times less than this value.

These metrics are crucial for understanding the performance and reliability of a system. For instance, if p99 is consistently high, it might indicate frequent, significant performance bottlenecks that could affect user experience or business operations.

On a Datadog dashboard, you would typically use specific visualizations like line charts, histograms, or summary statistics to display these metrics. Now, let's break down how to set up and interpret such a dashboard:

#### Triage
What the Question is Asking
Question: How to display p95 or p99 latency on a Datadog dashboard.
What the Interviewer is Probing: Understanding of APM (Application Performance Monitoring) concepts, familiarity with Datadog's features and visualizations.

#### Assumptions
You have basic knowledge of how to set up monitoring in Datadog.
The system being monitored has a performance metric that can be measured as response time.

#### First-Principles Solution
Collecting the Metric: Ensure you are collecting the response_time (or any equivalent) metrics for the service or endpoint in question.
Using Datadog Metrics: In Datadog, use the "Metrics" tab to define and configure your metric collection. For p95 and p99, you can use a percentile function on your time series data.
Define Metric Collection
Enable Percentile Calculation inDatadog Metrics
Visualization Setup: In the Datadog UI, navigate to "Dashboards" and select or create one. Add a line chart widget.
Adding p95 and p99 Lines:
Click on Add Query to add a new query to your dashboard.
Use the percentile function in the metric editor. For example, if you are using the response_time metric, set it up like this: {metric: "response_time", by: ["endpoint"], percentiles: [0.95, 0.99]}.
Enable Percentile Calculation inDatadog Metrics
Add Query Widget toDashboard

Customizing the Display:

Adjust the time range and resolution of your queries for better visibility.

#### Cons
ider adding annotations or alerts based on these values if needed.

Interpreting Results: Once the metrics are displayed, monitor trends over time. For example, p99 should generally be lower than p95, indicating fewer but more significant outliers.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: To display p95 or p99 latency on a Datadog dashboard, collect the relevant metrics, use percentile calculations in Datadog, and visualize them using line charts.

#### Common Mistakes
Not enabling percentile calculation correctly in metric definitions.
Using incorrect time ranges that obscure trends.
Failing to set up alerts or annotations for significant deviations.

If you were asked a follow-up question, you might be asked about how to handle noisy data, different ways to aggregate metrics, or integrating these visualizations with other Datadog features like APM and Logs.

**Code:**

```yaml
# Example Datadog Metric Query Configuration for p95 and p99
metric:
  - {name: "response_time", by: ["endpoint"], percentiles: [0.95, 0.99]}
```

**Flow:**

1. Define Metric Collection
2. Enable Percentile Calculation inDatadog Metrics
3. Add Query Widget toDashboard
4. Customize Time Range andResolution
5. Interpret Trends Over Time

