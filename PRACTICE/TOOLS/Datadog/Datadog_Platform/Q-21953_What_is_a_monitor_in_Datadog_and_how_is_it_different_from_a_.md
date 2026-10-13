---
type: "practice"
difficulty: "Basic"
---

# What is a monitor in Datadog and how is it different from a dashboard?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Database Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Database_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Datadog, a monitor is a critical component of their AIOps platform that helps you automatically detect and alert on issues across your IT infrastructure in real-time. Monitors are fundamentally different from dashboards, which are more static visualizations of historical data.

#### Triaging the Question
What is a monitor?: We need to explain what monitors do in Datadog.
What is a dashboard?: We should describe what dashboards provide and how they differ from monitors.
Key Assumptions: Monitors are part of an alerting system, while dashboards are about visualization.
From First Principles

Monitors:

Purpose: To continuously check the health of your systems and trigger alerts if something goes wrong.
Components:
Query Language: Uses a query language (e.g., Datadog Query Language, DQL) to define what data is being checked.
Conditions: Specifies criteria for when an alert should be triggered.
Alert Policy: Configures how alerts are sent and to whom.

Dashboards:

Purpose: To provide a visual summary of key metrics over time.
Components:
Widgets: Visual elements like graphs, tables, or traces that display specific metrics.
Layout: Arranged in rows and columns for easy viewing.
Data Sources: Can be linked to various data sources (e.g., logs, metrics) but do not trigger alerts.

#### Example Scenario

Imagine you have a web application running on multiple servers. You want to ensure that response times are within acceptable limits and immediately notified if they exceed certain thresholds. Here’s how you would use monitors:

Define the Monitor:

Use DQL to write a query that checks the average response time of your application endpoints.
Set conditions where an alert is triggered if the average exceeds 200ms for more than five consecutive minutes.

Create a Dashboard:

Add widgets to show the average and max response times, error rates, and other relevant metrics over the last hour or day.
Arrange these widgets in a layout that makes it easy to see at-a-glance if something is wrong.

#### Closing Summary

In summary, monitors in Datadog are active components designed for real-time alerting based on defined conditions, whereas dashboards provide static visualizations of historical data. Monitors help you stay proactive by immediately informing you when issues arise, while dashboards give you context and insights into what happened after an incident.

#### Common Mistakes/Follow-Ups
Mistake: Confusing the purpose of monitors with that of dashboards.

#### Follow-Up
Discussing scenarios where both are used together (e.g., creating a dashboard to provide context for monitor alerts).

**Code:**

```python
# Example DQL for response time monitor
response_time_monitor = "avg:my_application.response_time{env:prod} > 200m for 5m"
```

**Flow:**

1. Define Monitor
2. Set Conditions
3. Trigger Alert if Condition Met
4. Create Dashboard
5. Add Widgets
6. Rearrange Layout

