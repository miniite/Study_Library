---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would use tag-based scoping in monitors so one alert definition covers many services safely.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ APM and Trace Analytics](<../../../../TOOLS/Datadog/Datadog_Platform/APM_and_Trace_Analytics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how to use tag-based scoping in Datadog monitors, let's first understand the problem and then dive into a practical solution step by step.

TL;DR

Tag-based scoping allows you to create a single monitor definition that can apply to multiple services based on tags. This reduces redundancy and improves maintainability while ensuring alerts are safely scoped to the relevant services.

#### Triaging the Question

The question is asking how to leverage Datadog's tagging capabilities to manage alert definitions more efficiently for a variety of services. Specifically, it aims to:

Understand tag-based scoping
Learn about the benefits of using tags in monitors
Implement a solution that ensures alerts are correctly scoped without causing false positives or missing critical issues

#### First-Principles Solution
Step 1: Define Tags and Service Boundaries

In Datadog, services can be tagged with metadata such as environment (e.g., prod, staging), application name (e.g., webapp, backend), region, etc. These tags help in categorizing your services.

Tag-based scoping
Define service tags
Use tags in monitor definitions
Step 2: Create a Monitor Definition with Tags

Instead of creating separate monitors for each service, use the tags parameter to define which services should trigger the alert. For example:

YAML
apiVersion: datadoghq.com/v1beta1
kind: AlertmanagerConfig
metadata:
  name: webapp-alert
spec:
  receivers:
  - name: webapp-notify
    route:
      group_by: [service]
      routes:
        - receiver: "webapp-notification"
          match:
            service: ["webapp-*"]
Step 3: Ensure Proper Scoping and Filtering

When defining the monitor, use tags to filter which services should be monitored. This can include using regex patterns or specific tag values.

YAML
- definition:
    name: "Web Application CPU Usage Alert"
    query: avg(last_1h):sum:rate(process_cpu_time:{service: "webapp-*"}) by {host}
    alert_if: >
      > 0.85
Step 4: Test and Validate the Monitor

Test the monitor with sample data to ensure it behaves as expected. This involves:

Running queries on a small scale before full deployment
Monitoring for false positives or negatives
Define tags
Create monitor definition
Test monitors
Step 5: Implement and Monitor

After testing, deploy the monitor in your environment and continuously monitor its performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Key Takeaways: Using tag-based scoping helps in maintaining a single alert definition for multiple services while ensuring proper scoping.

#### Common Mistakes
Overlooking specific service tags, leading to alerts on unrelated services
Failing to test the monitor comprehensively before deployment
Not setting appropriate thresholds, causing either too many false positives or misses
Common Follow-Ups
How do you handle different environments (e.g., prod vs. staging)?
What are some best practices for managing tag-based scoping in Datadog?
How can you debug issues if an alert is not firing as expected?

**Code:**

```

apiVersion: datadoghq.com/v1beta1 kind: AlertmanagerConfig metadata: name: webapp-alert spec: receivers:

name: webapp-notify route: group_by: [service] routes: - receiver: "webapp-notification" match: service: ["webapp-*"]
CODE
Copy

```yaml
- definition:
    name: "Web Application CPU Usage Alert"
    query: avg(last_1h):sum:rate(process_cpu_time:{service: "webapp-*"}) by {host}
    alert_if: >
      > 0.85
```

**Flow:**

1. Define tags
2. Create monitor definition
3. Test monitors

