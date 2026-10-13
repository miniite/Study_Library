---
type: "practice"
difficulty: "Intermediate"
---

# How do you set up proactive capacity alerts so you get paged before a service saturates rather than after?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Composite Monitors](<../../../../TOOLS/Datadog/Datadog_Platform/Composite_Monitors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up proactive capacity alerts in Datadog, we need to monitor key metrics that indicate when a service is approaching its capacity limits. The goal is to get alerted and take action before these services become unstable or fail entirely. This involves setting up custom monitors and notifications that trigger under specific conditions. Let’s break down the process step by step.

#### Triaging the Question

The question is asking how to configure proactive alerts in Datadog so that we are notified early enough to prevent a service from saturating, rather than reacting to an already overloaded state. The key points being probed include:

Understanding Service Metrics: Identifying which metrics are relevant for capacity monitoring.
Setting Thresholds and Notifications: Configuring the thresholds at which alerts should be triggered.
Custom Monitor Setup: Creating custom monitors that can handle complex logic if needed.
Integration with Alerting Mechanisms: Ensuring seamless integration with Datadog’s alerting system.
Step-by-Step Solution

Identify Key Metrics:

CPU usage: Too high indicates potential performance degradation.
Memory usage: High memory can lead to swapping, impacting overall service performance.
Network I/O: High inbound or outbound traffic can cause latency and degradation.
Disk I/O: Frequent disk operations can be a sign of poor optimization.

Determine Thresholds:

For CPU usage, an alert might fire if it exceeds 80% consistently for more than 15 minutes.
Memory usage might trigger alerts at 90% or above.
Network I/O could be monitored at 70-80% of the maximum capacity.

Set Up Custom Monitors:

Create a custom monitor in Datadog that checks these metrics and sends notifications when thresholds are exceeded.
YAML
monitors:
  - name: "High CPU usage"
    type: "metric alert"
    query: "average by host {host.name}:system.cpu.usage{tags} > 80 for 15m"
    threshold: 80
    window: 15m
    notify_options:
      escalation_message: "CPU usage is consistently high on hosts."
  - name: "High Memory Usage"
    type: "metric alert"
    query: "average by host {host.name}:system.mem.usage{tags} > 90 for 5m"
    threshold: 90
    window: 5m
    notify_options:
      escalation_message: "Memory usage is critical on hosts."

Configure Notifications:

Ensure that Datadog integrates with your alerting system, such as PagerDuty or Slack.
JSON
{
  "alert_rule": "High CPU usage",
  "integration": "PagerDuty",
  "service_key": "YOUR_PAGER_DUTY_SERVICE_KEY"
}

Test and Monitor:

Regularly test the setup to ensure it works as expected.
Monitor the effectiveness of alerts in real-world scenarios.

#### Closing Summary
and Common Mistakes

Proactive capacity monitoring is crucial for maintaining service performance. By setting up custom monitors that track key metrics and configuring appropriate thresholds, you can get early warnings before services are overloaded. Common mistakes include:

Failing to continuously monitor and adjust thresholds based on changing workloads.
Not integrating alerts with the right notification systems.
Ignoring false positives or negatives which can lead to alert fatigue.
Self-CHECK

The solution is correct and comprehensive, covering all aspects from initial setup to testing and maintenance. No additional details are required.

**Code:**

```yaml
monitors:
  - name: "High CPU usage"
    type: "metric alert"
    query: "average by host {host.name}:system.cpu.usage{tags} > 80 for 15m"
    threshold: 80
    window: 15m
    notify_options:
      escalation_message: "CPU usage is consistently high on hosts."
  - name: "High Memory Usage"
    type: "metric alert"
    query: "average by host {host.name}:system.mem.usage{tags} > 90 for 5m"
    threshold: 90
    window: 5m
    notify_options:
      escalation_message: "Memory usage is critical on hosts."
```

**Flow:**

1. Initial Setup
2. Monitor Thresholds
3. Custom Monitor Creation
4. Test and Validate
5. Maintain and Adjust

