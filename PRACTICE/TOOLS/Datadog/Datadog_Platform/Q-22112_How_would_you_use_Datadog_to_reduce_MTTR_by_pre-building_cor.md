---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Datadog to reduce MTTR by pre-building correlated views for your top three recurring incident types?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Log Parsing and Notebooks](<../../../../TOOLS/Datadog/Datadog_Platform/Log_Parsing_and_Notebooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively leverage Datadog in reducing Mean Time To Resolution (MTTR) by pre-building correlated views, we need to focus on identifying and addressing common incident patterns. Here’s a step-by-step approach:

Firstly, understand that the goal is to create custom dashboards and alerts for recurring issues. By doing so, you can quickly pinpoint the source of problems before they escalate, allowing for faster resolution.

This task involves several key steps:

Identify Recurring Incidents: Analyze historical data in Datadog to identify top three incident types that cause frequent outages or performance degradations.
Correlate Metrics and Logs: Use Datadog’s powerful querying capabilities to correlate metrics, traces, and logs for these incidents.
Create Custom Views: Build visualizations and alerts that highlight critical components and patterns associated with each recurring incident type.
Automate Alerting and Notifications: Configure automated notifications and workflows based on the custom views.

Let's break down each step in detail:

Identify Recurring Incidents:
Use Datadog’s alert history to filter out the most common alerts over a set period, typically a month.
Review the incident notes and comments to understand the root causes of these issues.
Correlate Metrics and Logs:
For each identified incident type, use Datadog’s APM (Application Performance Monitoring) or metrics explorer to find correlations between different services, processes, and infrastructure components.
Trace logs from the same time period to identify patterns in user actions or system behavior that might lead to the incidents.
Create Custom Views:
For each incident type, create custom dashboards with relevant metrics, histograms, and traces.
Use Datadog’s top_events query to surface events related to these metrics.
Add annotations for known issues or maintenance windows that might be correlated with the incidents.
Automate Alerting and Notifications:
Set up custom alerts using Datadog's alert policies based on the specific patterns identified.
Integrate with a communication platform (like Slack) to send real-time notifications when these conditions are met.

By following this process, you can significantly reduce MTTR by providing front-line engineers with the necessary context and visibility to quickly address recurring incidents. This approach not only speeds up resolution times but also builds a more proactive monitoring culture.

#### Common mistakes
include:

Failing to analyze historical data thoroughly, leading to irrelevant or ineffective views.
Not correlating metrics and logs properly, which can result in missing critical information.
Overlooking the importance of automation, leaving manual steps that delay response times.

