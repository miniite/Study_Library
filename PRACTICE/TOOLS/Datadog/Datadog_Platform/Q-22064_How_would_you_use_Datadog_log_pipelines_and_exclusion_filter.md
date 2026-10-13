---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Datadog log pipelines and exclusion filters to cut indexing cost while keeping enough data for incident forensics?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Log Parsing and Notebooks](<../../../../TOOLS/Datadog/Datadog_Platform/Log_Parsing_and_Notebooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively manage indexing costs in Datadog without compromising on critical data for incident forensics, we can leverage Datadog's log pipelines and exclusion filters to filter out unnecessary logs while ensuring that key information is retained. In this answer, I will outline the steps and rationale behind using these features.

TL;DR
Understand the Logs: Identify which logs are critical for incident forensics.
Use Log Pipelines: Direct non-essential logs to cheaper storage tiers or external storage systems.
Apply Exclusion Filters: Exclude irrelevant log entries from indexing and storage without losing essential data.

#### Triage

The question is essentially asking how to optimize Datadog's log management while maintaining the integrity of critical incident-related information. The core idea involves understanding what logs are necessary for forensics, setting up pipelines to direct non-critical logs elsewhere, and applying exclusion filters to prune unnecessary data.

#### First-Principles Solution

Identify Critical Logs:

Determine which log entries are essential for incident forensics.

#### Cons
ider application logs (e.g., errors, warnings), system logs (e.g., network issues, authentication failures), and specific metrics that can help in diagnosing problems.

Set Up Log Pipelines:

Use Datadog's log pipelines to route non-critical logs to a cheaper storage tier or an external data warehouse.

#### Example
Route logs older than 90 days from the default datadog index to a cheaper S3 bucket via a pipeline.

Apply Exclusion Filters:

Configure exclusion filters in Datadog to remove redundant log entries that do not contribute to incident analysis.
For instance, exclude logs with specific error codes or patterns that are known to be non-urgent.

Monitor and Validate:

Continuously monitor the effectiveness of these configurations.
Ensure that critical logs are retained in the main index while non-critical data is managed efficiently.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing log pipelines and exclusion filters, we can significantly reduce storage costs without sacrificing the ability to perform thorough incident forensics. Key mistakes include failing to properly identify essential logs or over-filtering critical information. It's also important to regularly review and update these configurations as application needs change.

#### Common Mistakes
Over-Filtering: Excluding too much data, leading to loss of critical information.
Insufficient Monitoring: Not setting up alerts to detect issues in real-time.
Neglecting Data Retention Policies: Failing to plan for long-term data storage and compliance requirements.

#### Follow-Ups
Cost Analysis Tools:
Use Datadog’s built-in cost analysis tools to monitor and adjust your configurations based on actual costs.
Automated Configurations:

#### Cons
ider using Datadog's log management policies to automate the setup of pipelines and exclusion rules.

**Comparison:**

```
Log Type	Retention Policy	Storage Location
Application Errors	90 days + real-time alerting	Main Index
System Warnings	365 days	Cheaper S3 Bucket
Specific Metrics (e.g., CPU usage)	Real-time alerts	Main Index
```

**Code:**

```python
# Example Python script to create a pipeline in Datadog API
import requests

pipeline_response = requests.post(
    "https://api.datadoghq.com/api/v1/logs/pipeline",
    headers={
        "Content-Type": "application/json",
        "DD-API-KEY": "YOUR_API_KEY",
        "DD-MANAGE": "true"
    },
    json={
        "name": "my_pipeline",
        "source_type_name": "my_source",
        "index_name": "default",
        "target_index_name": "s3 cheaper storage",
        "data_retention_days": 90
    }
)
print(pipeline_response.json())
```

**Example input / output:**

```text
Input:
- Logs from application and system sources, including errors, warnings, and metrics.
- Specific patterns like error codes.

Output:
- Critical logs retained in the main index for forensics.
- Non-critical logs directed to cheaper storage or excluded entirely.
```

**Data:**

```json
{
    "name": "example_pipeline",
    "source_type_name": "app_logs",
    "index_name": "default",
    "target_index_name": "s3 cheaper storage",
    "data_retention_days": 90,
    "exclude_filters": [
        {
            "match": "error_code:404",
            "reason": "non-urgent"
        }
    ]
}
```

**Flow:**

1. Start
2. Identify Critical Logs
3. Set Up Log Pipelines
4. Apply Exclusion Filters
5. Monitor and Validate
6. End

