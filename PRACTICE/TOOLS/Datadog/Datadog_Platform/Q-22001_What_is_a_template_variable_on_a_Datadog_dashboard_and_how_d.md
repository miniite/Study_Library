---
type: "practice"
difficulty: "Intermediate"
---

# What is a template variable on a Datadog dashboard and how does it make dashboards reusable?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ APM and Trace Analytics](<../../../../TOOLS/Datadog/Datadog_Platform/APM_and_Trace_Analytics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Template variables in Datadog dashboards are dynamic placeholders that allow you to parameterize your visualizations, making them reusable across different data sources or contexts. Essentially, template variables enable flexibility and reusability by letting users select specific metrics or tags from a dropdown menu, which dynamically updates the dashboard's content.

To understand this concept, let's break down what Datadog is probing: first, the definition of template variables; second, their implementation in practice; and finally, how they contribute to overall dashboard functionality. I'll explain these step by step using simple terms that a fresher can grasp but with enough detail for an intermediate engineer.

Firstly, template variables in Datadog are typically used within panels on dashboards. They act as placeholders for metric names or tags, which get populated from the user's selection during runtime. For example, you might have a panel where the query is something like avg:my_metric{host:${host}}—here, ${host} acts as a template variable that the user can choose different values for.

Now let’s consider a practical scenario to illustrate how this works:

Imagine you're monitoring multiple servers in your data center. You want to create a dashboard where you can quickly switch between these servers without having to manually change the metric name each time. By using a template variable called ${host}, you can build a generic panel that updates based on which server you select.

Here's how we can set this up step by step:

Define the host variable in your dashboard settings.
Use the avg:my_metric{host:${host}} query in one of your panels.
When viewing the dashboard, you'll see a dropdown with all available host names (or any other data that can be used as a template).
Selecting a different host from the dropdown updates the metric displayed on the panel.

This approach not only simplifies maintenance and scaling but also enhances usability by reducing repetitive work for users.

In summary, template variables in Datadog are powerful tools for creating flexible and reusable dashboards that can handle dynamic data needs efficiently.

**Code:**

```json
{
  "template_variables": [
    {
      "name": "host",
      "default_value": "server1",
      "type": "HOST"
    }
  ],
  "panels": [
    {
      "title": "CPU Utilization by Host",
      "type": "timeseries",
      "metrics": [
        {
          "query": "avg:system.cpu.user{*}{host:${host}}",
          "display_mode": "line"
        }
      ]
    }
  ]
}
```

**Flow:**

1. Define Template Variable
2. Select Value
3. Update Dashboard withSelected Value
4. Display Updated Data on Panel

