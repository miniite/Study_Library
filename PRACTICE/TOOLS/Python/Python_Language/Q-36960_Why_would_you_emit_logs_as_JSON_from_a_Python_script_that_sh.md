---
type: "practice"
difficulty: "Intermediate"
---

# Why would you emit logs as JSON from a Python script that ships logs to a system like Elasticsearch or CloudWatch?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Log Filtering Scripts](<../../../../TOOLS/Python/Python_Language/Log_Filtering_Scripts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When developing a Python script that needs to ship logs to an external system such as Elasticsearch or CloudWatch, emitting logs in JSON format is advantageous. This approach provides several benefits, including structured data storage, easier parsing and querying, and better integration with logging tools.

#### Triaging the Question

The core of this question revolves around understanding why JSON is a preferred log format over plain text for shipping logs to centralized systems like Elasticsearch or CloudWatch. The key points being probed are:

Data Structure: How does structured data (JSON) differ from unstructured data?
Integration Capabilities: What advantages do structured logs offer in terms of integration with other tools and services?
Querying Efficiency: Why is JSON more efficient for querying and analyzing log data?
Step-by-Step Solution

Understanding Log Formats:

Plain Text Logs: These are human-readable but lack structure, making parsing and querying complex.
JSON Logs: These provide a structured format that can be easily parsed and queried by tools.

Structured vs Unstructured Data:

In plain text logs, each line is typically an independent log entry without context. This makes it harder to correlate related events.
JSON logs include key-value pairs, making the data more organized and easier to parse programmatically.

Advantages of JSON Logs:

Easier Parsing: Tools like Logstash or Fluentd can easily parse JSON logs into structured data, which is then indexed in Elasticsearch or stored in CloudWatch.
Facilitates Querying: With structured data, you can query specific fields using Elasticsearch's powerful query language. For example, filtering by status code or searching for errors.
Integration with Monitoring Tools: Logs stored as JSON can be easily ingested and visualized in monitoring dashboards, providing a richer set of insights.

#### Example
Use Case: Consider a scenario where you are deploying an application that handles user requests. If logs are emitted in plain text format:

CODE
[2023-10-05 16:47:08] ERROR - User request failed with status code 500

In contrast, using JSON would provide more detail:

JSON
{
    "timestamp": "2023-10-05T16:47:08",
    "level": "ERROR",
    "message": "User request failed with status code 500",
    "user_id": 123,
    "request_id": "abc-def",
    "status_code": 500
}

#### Closing Summary
and Common Mistakes

Emitting logs as JSON from a Python script is beneficial because it enables better integration, easier parsing, and more efficient querying. The primary mistake to avoid is emitting unstructured text logs that lack the necessary context for analysis and correlation.

Common Follow-Ups
How do you handle log rotation with JSON format?
What are some tools used to parse and visualize JSON logs in a DevOps environment?
Can you give an example of a scenario where plain text logs might be preferred over JSON?

**Code:**

```python
import json

def log_to_json(timestamp, level, message, user_id=None, request_id=None, status_code=None):
    log_entry = {
        "timestamp": timestamp,
        "level": level,
        "message": message,
    }

    if user_id:
        log_entry["user_id"] = user_id
    if request_id:
        log_entry["request_id"] = request_id
    if status_code:
        log_entry["status_code"] = status_code

    return json.dumps(log_entry)
```

**Example input / output:**

```text
# Example of a log entry in JSON format
{
    "timestamp": "2023-10-05T16:47:08",
    "level": "ERROR",
    "message": "User request failed with status code 500",
    "user_id": 123,
    "request_id": "abc-def",
    "status_code": 500
}
```

