---
type: "practice"
difficulty: "Intermediate"
---

# How would you route a Datadog alert to a specific Slack channel or PagerDuty on-call team?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Alert Automation and Runbooks](<../../../../TOOLS/Datadog/Datadog_Platform/Alert_Automation_and_Runbooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To route a Datadog alert to a specific Slack channel or PagerDuty on-call team, we need to integrate Datadog with these third-party services through their APIs and webhooks. This process involves creating alerts in Datadog, setting up the necessary configurations for Slack and PagerDuty, and then ensuring that Datadog sends appropriate notifications based on the alert conditions.

The key steps are:

Define Alert Conditions: Set up specific alert rules in Datadog to define when an alert should be triggered.
Configure Webhooks: Create webhooks in both Slack and PagerDuty to receive the alert data from Datadog.
Route Alerts: Use Datadog's alert suppression or routing features to direct alerts to the correct Slack channel or PagerDuty team.

Let’s break this down step by step:

TL;DR

We need to set up webhooks in both Slack and PagerDuty, configure Datadog alerts to trigger these webhooks, and route the alerts based on specific conditions.

#### Triaging the Question

The core of the question is about routing Datadog alerts. The interviewer is probing your understanding of:

How Datadog alert rules are configured.
The process of setting up webhooks in external services like Slack and PagerDuty.
Best practices for managing and routing alerts to ensure timely responses.
Step-by-Step Solution

Alert Conditions in Datadog:

Navigate to the Datadog UI and create or modify alert rules based on the specific conditions you want to monitor.
Define the metric, threshold, and other parameters that trigger an alert.

Setting Up Slack Webhooks:

In the Datadog UI, go to Settings > Integrations.
Locate the Slack integration and create a new webhook for the specific Slack channel you want to route alerts to.
Note down the webhook URL (e.g., https://hooks.slack.com/services/<TEAM_ID>/<CHANNEL_ID>/<TOKEN>).

Setting Up PagerDuty Webhooks:

In the Datadog UI, go to Settings > Integrations.
Locate the PagerDuty integration and create a new service or escalation policy for the on-call team you want to route alerts to.
Note down the webhook URL (e.g., https://events.pagerduty.com/v2/enqueue).

Routing Alerts in Datadog:

In the alert rule configuration, under Alerts, select the webhooks you created for Slack and PagerDuty.
Ensure that each alert condition has the correct webhook URL associated with it based on the target channel or team.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By setting up these configurations correctly, Datadog will trigger the appropriate notifications to both Slack and PagerDuty. This ensures that relevant teams get the necessary information quickly and can respond effectively.

#### Common Mistakes

Not defining alert conditions clearly enough.
Failing to test webhooks before relying on them in production.
Misconfiguring webhook URLs, leading to alerts not being sent at all or sent to incorrect channels/teams.

#### Follow-Ups
How would you manage alert suppression during maintenance windows?
What metrics should be monitored for the health of your integrations with Slack and PagerDuty?

**Comparison:**

```
Webhook	URL	Purpose
Slack	https://hooks.slack.com/services/<TEAM_ID>/<CHANNEL_ID>/<TOKEN>	Send alerts to specific Slack channel
PagerDuty	https://events.pagerduty.com/v2/enqueue	Trigger incident in PagerDuty on-call team
```

**Code:**

```
Example Python script to send an alert to a Slack channel using Datadog webhook

import requests

webhook_url = "https://hooks.slack.com/services/<TEAM_ID>/<CHANNEL_ID>/<TOKEN>" message = { "text": "Alert: CPU usage is high on server ABC.", "channel": "#alerts", }

response = requests.post(webhook_url, json=message) print(response.status_code)

CODE
Copy

```python
# Example Python script to send an alert to PagerDuty using Datadog webhook
import requests

webhook_url = "https://events.pagerduty.com/v2/enqueue"
message = {
    "service_key": "abcd1234",
    "payload": {
        "summary": "Alert: CPU usage is high on server ABC.",
        "incident_key": "cpu_high_abc",
        "event_action": "trigger",
        "source": "Datadog Alert",
        "severity": "critical",
        "custom_details": {"server_name": "ABC", "cpu_usage": 95},
    },
}

response = requests.post(webhook_url, json=message)
print(response.status_code)
```

**Example input / output:**

```text
Input:
- Datadog alert triggered due to high CPU usage on server ABC.

Output (Slack):
{
    "text": "Alert: CPU usage is high on server ABC.",
    "channel": "#alerts"
}

Response:
200 OK

Output (PagerDuty):
{
    "service_key": "abcd1234",
    "payload": {
        "summary": "Alert: CPU usage is high on server ABC.",
        "incident_key": "cpu_high_abc",
        "event_action": "trigger",
        "source": "Datadog Alert",
        "severity": "critical",
        "custom_details": {"server_name": "ABC", "cpu_usage": 95}
    }
}

Response:
200 OK
```

**Data:**

```json
{
  "webhook_url_slack": "https://hooks.slack.com/services/<TEAM_ID>/<CHANNEL_ID>/<TOKEN>",
  "webhook_url_pagerduty": "https://events.pagerduty.com/v2/enqueue"
}
```

**Flow:**

1. Datadog Alerts
2. Yes
3. No
4. Define Alert Conditions
5. Alert Triggers?
6. Send Slack Webhook
7. Send PagerDuty Webhook
8. No Action Required
9. Route to SpecificChannels/Teams

