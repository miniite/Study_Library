---
type: "practice"
difficulty: "Intermediate"
---

# How do you keep Datadog dashboards useful during an incident bridge when everyone is panic-clicking and adding widgets?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Alert Automation and Runbooks](<../../../../TOOLS/Datadog/Datadog_Platform/Alert_Automation_and_Runbooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure Datadog dashboards remain useful during a critical incident, we need to maintain clarity amidst the chaos of multiple stakeholders adding widgets. The core challenge is to balance the need for comprehensive monitoring with the risk of clutter and confusion. Key strategies include proactive dashboard design, real-time updates, and automated widget management.

#### Triage

The question asks how to keep Datadog dashboards useful during an incident bridge when many users are adding widgets. This probes your ability to manage dashboard complexity in a high-stress environment. Assumptions include that the team is using Datadog for monitoring, and there’s a need to ensure key metrics remain visible despite user pressure.

#### First-Principles Solution
Proactive Dashboard Design: Create predefined dashboards with essential widgets for different roles.
Real-Time Updates: Ensure real-time data visibility to quickly identify issues.
Automated Widget Management: Use Datadog's automation features to dynamically add or remove widgets based on thresholds.
Proactive Dashboard Design
Role-Based Dashboards: For developers, operations teams, and business stakeholders, create dashboards that show the most relevant metrics for each role.
Developers: Focus on logs, error rates, and code health.
Operations Teams: Monitor service uptime, response times, and infrastructure status.
Business Stakeholders: Provide key performance indicators (KPIs) and financial metrics.
Real-Time Updates
Alerting: Set up alerts to notify stakeholders of critical issues in real-time. Datadog's alerting system can be configured to send notifications via email, SMS, or Slack.
PYTHON
# Example Datadog alert configuration
dd_event = {
    "event": "Incident Alert",
    "alert_type": "error",
    "title": "Service A is experiencing high latency issues",
    "text": "The response time for Service A has exceeded the threshold. Please investigate.",
    "priority": "warning"
}
Automated Widget Management
Dynamic Widget Addition: Use Datadog's API to dynamically add widgets based on alert status or predefined thresholds.
PYTHON
# Example Datadog API call to add a widget
response = datadog_api.add_widget(
    dashboard_id=12345,
    title="Service B Response Time",
    query="avg:system.net.bytes.sent{*} by {host}",
    type="timeseries"
)

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these strategies, you can ensure that dashboards remain useful even during a high-stress incident. Key mistakes include failing to anticipate user behavior or not having clear roles for dashboard access. Follow-ups might involve discussing how to handle false positives in alerts or the need for frequent updates to dashboards based on evolving incidents.

**Comparison:**

```
Role	Essential Metrics
Developers	Logs, error rates, code health
Operations Teams	Service uptime, response times, infrastructure status
Business Stakeholders	KPIs, financial metrics
```

**Code:**

```python
# Example Datadog API call to add a widget
import datadog_api

def add_widget(dashboard_id, title, query, type):
    response = datadog_api.add_widget(
        dashboard_id=dashboard_id,
        title=title,
        query=query,
        type=type
    )
    return response

response = add_widget(12345, "Service B Response Time", "avg:system.net.bytes.sent{*} by {host}", "timeseries")
```

**Example input / output:**

```text
Input:
{
  "event": "Incident Alert",
  "alert_type": "error",
  "title": "Service A is experiencing high latency issues",
  "text": "The response time for Service A has exceeded the threshold. Please investigate.",
  "priority": "warning"
}

Output:
{
  "status": "success",
  "message": "Alert event sent successfully."
}
```

**Data:**

```json
{
  "dd_event": {
    "event": "Incident Alert",
    "alert_type": "error",
    "title": "Service A is experiencing high latency issues",
    "text": "The response time for Service A has exceeded the threshold. Please investigate.",
    "priority": "warning"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Start
8. Is Dashboard ProactivelyDesigned?
9. Define Role-Based Dashboards
10. Is Real-Time Data VisibilityImportant?
11. Enable Real-Time Updates
12. Do You Need AutomatedWidget Management?
13. Implement Dynamic WidgetAddition
14. End

