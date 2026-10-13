---
type: "practice"
difficulty: "Intermediate"
---

# How do you use Datadog Live Tail responsibly during an incident without missing indexed data you will need later?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Database Monitoring](<../../../../TOOLS/Datadog/Datadog_Platform/Database_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively use Datadog Live Tail responsibly during an incident, we need to ensure that no critical information is missed while maintaining a balance between real-time visibility and long-term data retention. This involves understanding how Live Tail works, its limitations, and best practices for filtering and archiving relevant logs.

#### Triaging the Question

The question revolves around using Datadog's Live Tail feature during an incident to monitor streaming log data in real time without missing indexed data that could be crucial later on. The interviewer is probing your understanding of:

How Live Tail works and its limitations.
Best practices for filtering and managing logs during incidents.
Strategies to ensure you capture all necessary data while maintaining responsiveness.

#### First Principles Solution
1. Understanding Datadog Live Tail

Live Tail in Datadog streams log data from the source in real time, providing immediate visibility into ongoing events. However, it has some limitations:

It might not retain logs for long periods.
There can be delays or drops in streaming due to network issues.
2. Setting Up Filtering Rules

To ensure you don't miss critical information:

Define Relevant Filters: Use Datadog's filtering capabilities to focus on specific log sources that are relevant during the incident. For example, filter by service name, error codes, or keywords like "error," "critical," or stack traces.
Implement Dynamic Filters: Create dynamic filters based on real-time alerts or changes in the system state. This can help you react quickly to new issues as they arise.
3. Archiving Logs Responsibly

To avoid missing indexed data later:

Set Up Log Retention Policies: Configure Datadog to retain logs for a longer period, such as 90 days, so that you can go back and review them if needed.
Use Archive Buckets: If your incident involves critical operations or high-risk systems, consider setting up dedicated archive buckets in Datadog to store important log data separately.
4. Monitoring and Validation

To ensure completeness and accuracy:

Monitor Stream Quality: Keep an eye on the quality of Live Tail streams. Use Datadog's monitoring tools to check for any drops or delays.
Manual Reviews: During critical incidents, manually review logs periodically to ensure no important data is missed.

#### Closing Summary

In summary, using Datadog Live Tail responsibly during an incident involves setting up appropriate filtering and archiving strategies to balance real-time visibility with long-term data retention. By defining relevant filters, configuring log retention policies, and regularly validating the stream quality, you can minimize the risk of missing critical indexed data.

#### Common Mistakes
and Follow-Ups
Failing to Set Up Filters: Not setting up specific filters for the incident can lead to irrelevant noise in your live stream.
Over-reliance on Live Tail: Relying solely on Live Tail without setting up proper archiving policies can result in missing crucial data that needs to be reviewed later.

#### Follow-ups
might include:

Discussing specific scenarios where certain filters or retention policies are most useful.
Exploring advanced features like log aggregation and correlation for better incident response.
Addressing potential cost implications of longer log retention periods.

**Comparison:**

```
Filter Type	Description
Service	Filters logs by the service name. For example, service:"example-service".
Level	Filters logs based on severity level. For example, level:error for critical errors.
Retention Policy	Days to Retain Logs
Default	30
Extended	90
```

**Code:**

```python
# Example code to set up a Datadog filter using the Datadog Python API
from datadog import initialize, api

options = {
    'api_key': '<API_KEY>',
    'app_key': '<APP_KEY>'
}

initialize(**options)

response = api.Filter.create(
    query='service:"example-service" level:error'
)
print(response)
```

**Example input / output:**

```
Input: Define a filter query for logs from the example-service with error severity.
Output: Datadog returns a filter object with the specified name and query.
JSON
Copy
{
  "filter_id": "1234567890",
  "query": "service:\"example-service\" level:error",
  "name": "Example Service Error Logs"
}
```

**Data:**

```json
{
  "filters": [
    {
      "query": "service:\"example-service\" level:error",
      "name": "Example Service Error Logs"
    }
  ]
}
```

**Flow:**

1. Define Relevant Filters
2. Set Up Log Retention Policies
3. Monitor Stream Quality
4. Review Logs Periodically
5. Maintain Live Tail Responsibly

