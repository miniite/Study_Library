---
type: "practice"
difficulty: "Basic"
---

# What is a timeboard versus a screenboard in Datadog dashboards?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Datadog, both Timeboards and Screenboards are types of dashboards used to visualize data, but they serve different purposes and have distinct functionalities.

Firstly, a Timeboard in Datadog is essentially a dynamic dashboard that allows you to create time-series graphs. You can add multiple widgets such as line charts, bar charts, or histograms, each representing various metrics over time. Timeboards are particularly useful for monitoring and analyzing trends, anomalies, and performance over different periods.

On the other hand, a Screenboard in Datadog is more like a static dashboard that you can use to organize and display multiple widgets, including timeboards, but also text blocks, tables, and even external images. Screenboards are ideal for creating comprehensive reports or presentations where you need to combine different types of data visualizations.

To better understand the differences, let's consider an analogy: Imagine a Timeboard as a moving film where each frame represents a snapshot in time, allowing you to observe how metrics change over time. A Screenboard, by contrast, is like a single photograph or composite image that includes various elements—such as text explaining the situation, tables showing detailed data, and multiple moving films (timeboards) all combined into one cohesive view.

In Datadog, these dashboards are created using the Dashboard Designer, where you can customize the layout, add widgets, and configure visualizations. For example, a Timeboard might be configured with several line charts to monitor CPU usage over different hours of the day, while a Screenboard could contain a timeboard for CPU usage, text blocks explaining the context, and tables showing detailed logs.

The key assumptions here are that both dashboards (Timeboards and Screenboards) are part of Datadog's Dashboard Designer, which is the primary interface for creating and managing visualizations. Users can tailor these dashboards to fit their specific needs by adding different types of widgets and configuring them according to their requirements.

In summary, Timeboards focus on dynamic time-series visualization, while Screenboards provide a more flexible way to organize multiple types of data visualizations in one place.

**Comparison:**

```
Widget Type	Timeboard	Screenboard
Purpose	Dynamic time-series visualization	Flexible organization of multiple data visualizations
Examples	Line charts, bar charts	Text blocks, tables, external images
```

**Data:**

```text
{
  "timeboard": {
    "description": "A dashboard that displays time-series data over different periods.",
    "widgets": [
      {"type": "line_chart", "title": "CPU Usage by Hour"},
      {"type": "bar_chart", "title": "Memory Usage By Service"}
    ]
  },
  "screenboard": {
    "description": "A dashboard that combines various types of widgets, including timeboards.",
    "widgets": [
      {"type": "timeboard", "title": "CPU Usage by Hour"},
      {"type": "text_block", "content": "Performance report for the day."},
      {"type": "table", "title": "Detailed Logs"}
    ]
  }
}
```

