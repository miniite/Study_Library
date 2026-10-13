---
type: "practice"
difficulty: "Intermediate"
---

# What is your approach to alert routing so severity maps to the right escalation policy and low-signal alerts never wake anyone up?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Alert Automation and Runbooks](<../../../../TOOLS/Datadog/Datadog_Platform/Alert_Automation_and_Runbooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively manage alert routing in Datadog, ensuring that severity maps to the right escalation policies while minimizing unnecessary disruptions, we need a robust system that prioritizes alert severity and implements tiered escalations. Here’s how I would approach this:

Firstly, it's important to understand that Datadog provides extensive tools for managing alerts and customizing their behaviors, such as setting up severity levels, escalation policies, and notifications. The core idea is to design a routing system that dynamically routes alerts based on predefined rules, ensuring low-severity alerts are handled differently from critical ones.

We will triage the question by breaking it down into several components:

Severity Levels: Understanding and defining different alert severities (e.g., INFO, WARNING, CRITICAL).
Escalation Policies: Setting up policies that determine who gets notified at what stage.
Notification Methods: Configuring email, SMS, or other notification methods based on severity.

Now, let's walk through the solution step-by-step:

Define Severity Levels: We need to establish a clear hierarchy of alert severities. For instance, INFO alerts might be for system maintenance notifications, WARNING could indicate potential issues needing attention, and CRITICAL would involve immediate action.
Create Escalation Policies: Each severity level should have corresponding escalation policies. These policies define who gets notified at each stage—initially a bot or monitoring tool, then an on-call engineer, and finally a broader team if the issue persists.
Implement Routing Logic: Using Datadog’s API, we can route alerts to specific users or teams based on their severity. For example, CRITICAL alerts might be routed directly to senior engineers via SMS, while lower-severity alerts could go to monitoring tools without any human intervention.

To illustrate this, let's consider a practical scenario:

Alert A: A WARNING alert indicating a performance degradation in the database.

Initial notification: Datadog’s internal monitoring tool

#### Follow-up
Notification to on-call engineers if no resolution within 5 minutes

Alert B: A CRITICAL alert due to a service outage.

Immediate SMS notification to the primary on-call engineer

#### Follow-up
Notification to the entire engineering team if not resolved in an hour

This approach ensures that critical issues are handled promptly, while low-severity alerts do not disrupt everyone’s sleep or productivity.

**Comparison:**

```
Severity Level	Initial Notification	Follow-Up Actions
CRITICAL	Immediate SMS	Notify entire team after 1800 seconds if unresolved
WARNING	Monitoring tool	Notify on-call engineers after 300 seconds if not improved
```

**Code:**

```python
# Example Python script for defining severity levels and routing logic
def route_alert(alert):
    if alert['severity'] == 'CRITICAL':
        notify_oncall_engineer('SMS')
        if not check_resolution(1800):
            notify_entire_team('Email')
    elif alert['severity'] == 'WARNING':
        notify_monitoring_tool()
        if not check_improvement(300):
            notify_oncall_engineer('Email')

def notify_oncall_engineer(method='SMS'):
    # Implement notification logic
    pass

def notify_monitoring_tool():
    # Implement notification to monitoring tool
    pass

def notify_entire_team(method='Email'):
    # Implement notification logic
    pass

def check_resolution(timeout=1800):
    # Check if issue is resolved within timeout
    return True  # Example implementation
```

**Example input / output:**

```text
Input: Alert A (WARNING) with performance degradation in the database.
Output: Monitoring tool notified; no human intervention required for next 5 minutes.

Input: Alert B (CRITICAL) due to service outage.
Output: On-call engineer notified by SMS; if unresolved, entire team notified via email within 1800 seconds.
```

**Data:**

```yaml
# Sample YAML configuration for Datadog alert routing
escalation_policy:
  - name: critical-escalation
    notify_on:
      - "SMS": [oncall-engineer]
      - "Email": [entire-team] if not resolved within 1800 seconds

alert_rule:
  severity: CRITICAL
  on_call_engineers: true
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. CRITICAL Alert
8. Is Resolved?
9. Alert Resolved
10. Notify Entire Team?
11. Send Email to Entire Team
12. Check Resolution in 5Minutes?
13. Alert Resolved
14. Notify On-call Engineer bySMS

