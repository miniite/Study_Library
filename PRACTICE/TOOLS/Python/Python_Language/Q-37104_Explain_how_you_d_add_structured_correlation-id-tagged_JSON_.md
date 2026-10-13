---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd add structured, correlation-id-tagged JSON logging to a Python automation pipeline so a single deploy's logs across multiple subprocess and API calls can be traced in CloudWatch or Loki.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Request Logging for Models](<../../../../TOOLS/Python/Python_Language/Request_Logging_for_Models.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add structured, correlation-id-tagged JSON logging to a Python automation pipeline, we need to ensure that each log entry contains a unique identifier (correlation ID) that helps trace logs across subprocesses and API calls. This way, when issues arise during deployment, the logs can be easily correlated and analyzed in services like AWS CloudWatch or Grafana Loki.

The key steps are:

Generate a Unique Correlation ID: Ensure each log entry has a unique identifier.
Structure Log Entries as JSON: Make sure all log entries are structured in JSON format for easy parsing and querying.
Propagate the Correlation ID: Pass this ID through subprocesses and API calls to maintain traceability.

Let's break down these steps using first principles:

Step-by-Step Solution
Generate a Unique Correlation ID:
Use a UUID or timestamp-based approach for generating unique IDs.
Structure Log Entries as JSON:
Format logs in a consistent, structured manner (key-value pairs).
Propagate the Correlation ID:
Pass the ID through environment variables, function parameters, or context objects.

#### Implementation
Details
Step 1: Generate Unique Correlation IDs

We can use Python's uuid module to generate unique identifiers.

PYTHON
import uuid

def generate_correlation_id():
    return str(uuid.uuid4())
Step 2: Structure Log Entries as JSON

Define a logging function that formats logs in JSON:

PYTHON
import json
from datetime import datetime

def log(message, level="INFO", correlation_id=None):
    timestamp = datetime.utcnow().isoformat()
    if not correlation_id:
        correlation_id = generate_correlation_id()

    log_entry = {
        "timestamp": timestamp,
        "correlation_id": correlation_id,
        "level": level,
        "message": message
    }

    print(json.dumps(log_entry))
Step 3: Propagate the Correlation ID

Ensure that subprocesses and API calls receive and use this ID. We can do this by passing it through environment variables or function parameters.

PYTHON
def subprocess_task(correlation_id):
    # Set the correlation ID in the environment
    os.environ["CORRELATION_ID"] = correlation_id

    # Log within the subprocess
    log("Subprocess started", correlation_id=correlation_id)

    # Perform tasks...
    log("Task completed", correlation_id=correlation_id)

def api_call(correlation_id):
    # Use the correlation ID in API requests
    headers = {"x-correlation-id": correlation_id}
    response = requests.get(f"https://api.example.com/endpoint", headers=headers)

    # Log within the API call
    log("API response received", level="DEBUG", correlation_id=correlation_id)
Integration with CloudWatch or Loki

When deploying logs to AWS CloudWatch, ensure that your logging setup adheres to its expected format and policies. For Grafana Loki, which uses line protocol, you might need to adjust the JSON structure slightly.

#### Common Mistakes
and Follow-Ups
Lack of Correlation ID Propagation: Failing to pass down the correlation ID through subprocesses and API calls can result in untraceable logs.
Unstructured Logging: Using plain text logs without a structured format can complicate log analysis.
Incorrect Timestamp Handling: Ensure timestamps are correctly formatted for consistent logging.

#### Example
of Integration with AWS CloudWatch

To send these JSON logs to CloudWatch, you might use boto3 or other libraries:

PYTHON
import boto3

def send_to_cloudwatch(correlation_id, message):
    cloudwatch = boto3.client('logs')

    log_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "correlation_id": correlation_id,
        "level": "INFO",
        "message": message
    }

    response = cloudwatch.put_log_events(
        logGroupName="/aws/lambda/my_lambda_function",
        logStreamName="log_stream_name",
        logEvents=[{
            'timestamp': int(datetime.utcnow().strftime('%s')) * 1000,
            'message': json.dumps(log_entry)
        }]
    )

**Comparison:**

| Level | Message              |
|-------|----------------------|
| INFO  | Starting subprocess  |
| INFO  | Task completed       |

**Code:**

```python
import uuid
import os
from datetime import datetime
import requests

def generate_correlation_id():
    return str(uuid.uuid4())

def log(message, level="INFO", correlation_id=None):
    timestamp = datetime.utcnow().isoformat()
    if not correlation_id:
        correlation_id = generate_correlation_id()

    log_entry = {
        "timestamp": timestamp,
        "correlation_id": correlation_id,
        "level": level,
        "message": message
    }

    print(json.dumps(log_entry))

def subprocess_task(correlation_id):
    os.environ["CORRELATION_ID"] = correlation_id

    log("Subprocess started", correlation_id=correlation_id)

    # Perform tasks...
    log("Task completed", correlation_id=correlation_id)

def api_call(correlation_id):
    headers = {"x-correlation-id": correlation_id}
    response = requests.get(f"https://api.example.com/endpoint", headers=headers)

    log("API response received", level="DEBUG", correlation_id=correlation_id)

def send_to_cloudwatch(correlation_id, message):
    cloudwatch = boto3.client('logs')

    log_entry = {
        "timestamp": datetime.utcnow().isoformat(),
        "correlation_id": correlation_id,
        "level": "INFO",
        "message": message
    }

    response = cloudwatch.put_log_events(
        logGroupName="/aws/lambda/my_lambda_function",
        logStreamName="log_stream_name",
        logEvents=[{
            'timestamp': int(datetime.utcnow().strftime('%s')) * 1000,
            'message': json.dumps(log_entry)
        }]
    )
```

**Example input / output:**

```text
Input:
subprocess_task("7d2e3b5c-4f69-45c8-ba7e-c0b5d65da1de")

Output:
{"timestamp": "2023-10-01T14:30:00.123Z", "correlation_id": "7d2e3b5c-4f69-45c8-ba7e-c0b5d65da1de", "level": "INFO", "message": "Subprocess started"}

{"timestamp": "2023-10-01T14:30:00.123Z", "correlation_id": "7d2e3b5c-4f69-45c8-ba7e-c0b5d65da1de", "level": "INFO", "message": "Task completed"}
```

**Data:**

```json
{
    "correlation_id": "7d2e3b5c-4f69-45c8-ba7e-c0b5d65da1de",
    "timestamp": "2023-10-01T14:30:00.123Z",
    "level": "INFO",
    "message": "Starting subprocess"
}
```

**Flow:**

1. Yes
2. No
3. Generate Correlation ID
4. Log Entry
5. Is Correlation ID Needed?
6. Set Environment Variable
7. Call Subprocess Task/API Call
8. Proceed with Log Entry
9. Subprocess Executes
10. Subprocess Logs Completion

