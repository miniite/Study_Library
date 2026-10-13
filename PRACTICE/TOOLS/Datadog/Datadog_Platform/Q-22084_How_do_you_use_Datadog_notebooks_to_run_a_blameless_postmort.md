---
type: "practice"
difficulty: "Intermediate"
---

# How do you use Datadog notebooks to run a blameless postmortem that others can reproduce from the data?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Log Parsing and Notebooks](<../../../../TOOLS/Datadog/Datadog_Platform/Log_Parsing_and_Notebooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we will explore how to leverage Datadog notebooks to conduct a blameless postmortem in an AIOps context. The key idea is to meticulously document and analyze the incident from all perspectives, ensuring that everyone involved has clear visibility into the root cause and contributing factors. This approach not only helps in resolving current issues but also prevents future occurrences by sharing learnings.

Datadog notebooks are a powerful feature within Datadog for data visualization and analysis. They allow you to combine code, visualizations, and markdown in one place, making them ideal for postmortem analysis. Here’s how we can use them effectively:

Understanding the Postmortem Process: The primary goal of a blameless postmortem is to identify the root cause(s) of an incident without assigning blame. This process involves collecting relevant data, analyzing it, and documenting findings in a way that all stakeholders can understand.

#### Assumptions

We assume access to Datadog’s API for querying raw data.
Notebook users are familiar with basic SQL and Datadog’s query language (Flux).
The team has defined key performance indicators (KPIs) relevant to the application or system in question.

Step-by-Step Solution:

1. Data Collection

Collect all necessary data from different sources, including logs, metrics, and traces, using Datadog’s API or web interface. For example:

SQL
// SQL-like query for metric data
from("aws.cloudwatch.log_group")
|where("message contains 'error'")
|sort降序 by timestamp
2. Data Exploration

Use Datadog notebooks to visualize and explore the collected data. Start with basic time series analysis, then move on to more complex queries:

DATADOG
// Example of a timeline visualization
timeline
   |query("service:myapp")
   |stat("count(*)")
3. Root Cause Analysis

Identify patterns or anomalies that might indicate the root cause. For instance, you could correlate metrics with logs to see if there are specific conditions leading up to the incident:

DATADOG
// Correlating metrics and logs
index("/logs/myapp/")
|json()
|where("request.status_code == 500")
4. Documentation

Document your findings in a clear, concise manner using markdown within the notebook. Include visualizations and explanations of what each metric or log entry signifies:

MARKDOWN
## Timeline of the Incident
The following timeline shows the increase in errors around 14:30 UTC.

![] (https://example.com/timeline.png)

## Key Observations
- The number of failed requests spiked around this time.
- Logs indicate a specific user action led to the failure.
5. Reproducibility

Ensure that the notebook is structured in such a way that others can easily replicate your analysis:

DATADOG
// Example query to show how logs correlate with metrics
index("/logs/myapp/")
|json()
|where("request.status_code == 500")
|sort降序 by timestamp

#### Closing Summary
and Common Mistakes/Follow-Ups

Summary: By using Datadog notebooks, we can conduct a blameless postmortem that is both comprehensive and reproducible. This ensures that everyone involved has clear visibility into the incident, leading to better learning outcomes.

#### Common Mistakes

Not documenting the steps clearly: Ensure each step in your analysis is well-documented with explanations.
Overlooking data quality: Poorly formatted or missing logs can mislead your analysis. Always verify and clean your data before analysis.
Focusing only on metrics without context: Logs provide critical context that metrics alone cannot offer. Use both to get a full picture.

#### Follow-Ups
After the postmortem, update relevant dashboards and documentation with new findings. This helps in continuous improvement of the system.

**Comparison:**

```
2023-09-25
2023-09-25
2023-09-26
2023-09-26
2023-09-27
2023-09-27
2023-09-28
2023-09-28
2023-09-29
2023-09-29
2023-09-30
2023-09-30
2023-10-01
M1
M2
Correlation of Metrics and Logs
Example Timeline for Incident Analysis
```

**Code:**

```
DATADOG
Copy
// Example queries for notebook
from("aws.cloudwatch.log_group")
|where("message contains 'error'")
|sort降序 by timestamp

timeline
   |query("service:myapp")
   |stat("count(*)")

index("/logs/myapp/")
|json()
|where("request.status_code == 500")
```

**Example input / output:**

```text
## Timeline of the Incident
The following timeline shows the increase in errors around 14:30 UTC.

![](https://example.com/timeline.png)

## Key Observations
- The number of failed requests spiked around this time.
- Logs indicate a specific user action led to the failure.
```

**Data:**

```json
{
  "metrics": [
    {
      "name": "service:myapp",
      "time_range": "1 day ago to now"
    }
  ],
  "logs": [
    {
      "query": "/logs/myapp/",
      "filter": {"request.status_code == 500"},
      "sort_by": "-timestamp"
    }
  ]
}
```

**Flow:**

1. Collect Data
2. Explore Data
3. Analyze Root Causes
4. Document Findings
5. Ensure Reproducibility
6. Review and Improve System

