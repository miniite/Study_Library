---
type: "practice"
difficulty: "Intermediate"
---

# What is a service check in Datadog and how is it different from a metric?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Metrics vs Logs Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Metrics_vs_Logs_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what a service check in Datadog is, let's start with a brief overview. Service checks are custom scripts or commands that you can run to monitor specific aspects of your infrastructure, applications, and services directly from within the Datadog platform. They are different from metrics because they provide binary outcomes: success or failure, rather than numeric values.

In triage mode, let's break down what a service check is and how it differs from a metric:

Definition: A service check in Datadog is essentially an external command or script that you write to monitor your infrastructure or applications. It can be anything from checking the response time of a web server, verifying database connection status, to more complex checks like ensuring specific files exist on disk.

Purpose:

Binary Output: Service checks return one of three states: pass, warning, or critical. This makes them ideal for monitoring services that can be in a known good state or not.
Flexibility and Customization: You have complete control over the logic, allowing you to write custom scripts tailored to your specific needs.

Comparison with Metrics:

Metrics vs. Service Checks: Metrics are numeric values (like response times, memory usage) that provide quantitative data about system performance or application status. They are collected by Datadog agents and can be visualized on graphs, alerting rules, and dashboards.
Binary Outcomes: Service checks offer a binary state which is more about checking the presence of services rather than their exact values.

#### First-Principles Solution

Service Check Workflow: A service check runs on the Datadog agent or via an API call, and it can be scheduled to run at regular intervals.
Writing a Service Check: You would write a Python script (or any other language supported by Datadog) that performs your monitoring task. This could involve pinging a server, checking for file existence, or even interacting with a database.

#### Example

PYTHON
import requests

def check_service():
    url = "http://example.com"
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return "pass"
        else:
            return "critical"
    except Exception as e:
        return f"warning: {str(e)}"

#### Closing Summary
In summary, service checks in Datadog are powerful tools for monitoring services with binary outcomes, providing a different perspective compared to the numeric data provided by metrics.

#### Common Mistakes/Follow-Ups

Misuse of Metrics: One common mistake is using a metric when a service check would be more appropriate.
Ignoring Contextual Information: Sometimes, users might not realize that service checks can carry contextual information beyond just the state (pass, warning, critical).

**Code:**

```python
import requests

def check_service():
    url = "http://example.com"
    try:
        response = requests.get(url)
        if response.status_code == 200:
            return "pass"
        else:
            return "critical"
    except Exception as e:
        return f"warning: {str(e)}"
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Run Service Check] --> B{Is URL reachable?};
4. B -->|Yes| C[Return "pass"];
5. B -->|No| D[Return "critical"];
6. D --> E[Handle Exception and return "warning"];

