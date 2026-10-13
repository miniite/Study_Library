---
type: "practice"
difficulty: "Intermediate"
---

# What is Log Analytics workspace in Azure and what is KQL used for?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Azure SQL and App Service Deployments](<../../../../TOOLS/Azure/Azure_Platform/Azure_SQL_and_App_Service_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Log Analytics workspace in Azure is a centralized repository where you can store, query, visualize, and analyze your log data from various sources. It's like a big digital notebook that keeps track of everything happening in your infrastructure.

KQL (Kusto Query Language) is the powerful querying language used to extract insights from this log data. Think of it as a supercharged version of SQL, but designed specifically for time-series data and log analysis.

To understand the question fully, let's break down what each term means and how they work together.

**Comparison:**

| Column Name | Description |
|-------------|-------------|
| TimeGenerated | Timestamp when the event occurred |
| Category     | Type of event (e.g., FailedLogin) |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Log Analytics Workspace] --> B{What is it?};
    B --> C[Centralized repository for log data];
    B --> D[Tools for analysis and visualization];
    B --> E[Supports various data sources];

    F[KQL (Kusto Query Language)] --> G{What does KQL do?};
    G --> H[Allows querying time-series data];
    G --> I[Suitable for log analytics];
    G --> J[More powerful than SQL for logs];
```

**Example input / output:**

```

To see how KQL works, let's consider a simple query to find the number of failed login attempts in the past hour:

KQL
Copy
let FailedLogins = ActivityLogs
| where Category == "FailedLogin" and TimeGenerated >= ago(1h);
FailedLogins
| summarize Count() by bin(TimeGenerated, 1h)

This query filters for FailedLogin events within the last hour and then summarizes them hourly.
```

**Data:**

```json
{
  "LogAnalyticsWorkspace": {
    "Description": "Centralized repository for storing and analyzing log data from various sources.",
    "Features": [
      "Data ingestion",
      "Real-time analytics",
      "Visualization tools",
      "Integration with other Azure services"
    ]
  },
  "KQL": {
    "Description": "Query language designed for time-series and log data analysis in Log Analytics Workspace.",
    "KeyFeatures": [
      "Time-based queries",
      "Complex aggregations and filtering",
      "Performance optimization techniques"
    ]
  }
}
```

**Flow:**

1. Log Data
2. Ingestion into Log AnalyticsWorkspace
3. KQL Query Execution
4. Results and Insights

