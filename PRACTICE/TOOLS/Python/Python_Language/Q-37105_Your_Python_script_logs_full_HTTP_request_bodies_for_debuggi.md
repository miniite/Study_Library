---
type: "practice"
difficulty: "Intermediate"
---

# Your Python script logs full HTTP request bodies for debugging and it leaked an API token into CloudWatch. How would you build a logging filter that redacts secrets and PII before anything is emitted?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to build a Python script that can intercept HTTP request bodies, identify sensitive information such as API tokens or personally identifiable information (PII), and redact them before logging. The key steps involve parsing the request body, identifying patterns for secrets and PII, replacing those with placeholders, and then emitting the sanitized data.

#### Triaging the Question

The question is asking how to implement a custom logging filter in Python that can scan and sanitize sensitive information from HTTP request bodies before they are logged. The interviewer is probing your understanding of regular expressions, string manipulation, and possibly configuration management within a logging framework like AWS CloudWatch.

We'll assume the following:

We're using the boto3 library to interact with CloudWatch.
The script will intercept requests and modify their request bodies before they are logged.

#### First Principles Solution
Identify Sensitive Information: Define regular expressions or patterns that match API tokens, PII, etc.
Capture Request Bodies: Use a logging framework's interception mechanism to capture HTTP request bodies.
Sanitize Data: Apply the defined patterns to redact sensitive information.
Log Sanitized Data: Emit the sanitized data to CloudWatch.

#### Implementation

We'll use boto3 for interacting with CloudWatch and define a simple regex pattern to identify tokens. We’ll also use Python's re module for string manipulation.

PYTHON
import boto3
import re

# Define regex patterns for identifying API tokens or PII
token_pattern = re.compile(r'\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b', re.IGNORECASE)
api_token_pattern = re.compile(r'your_api_token_here')

def redact_request_body(request_body):
    # Redact email addresses
    request_body = token_pattern.sub('<REDACTED_EMAIL>', request_body)

    # Redact API tokens
    request_body = api_token_pattern.sub('<REDACTED_TOKEN>', request_body)

    return request_body

# Function to log sanitized data to CloudWatch
def log_to_cloudwatch(sanitized_request):
    client = boto3.client('logs')

    # Define the log group and log stream name
    log_group_name = 'your-log-group-name'
    log_stream_name = f'log-stream-{sanitized_request["timestamp"]}'

    # Emit the sanitized data to CloudWatch Logs
    response = client.put_log_events(
        logGroupName=log_group_name,
        logStreamName=log_stream_name,
        logEvents=[
            {
                'timestamp': int(sanitized_request['timestamp']),
                'message': f"HTTP Request: {sanitized_request['request_body']}"
            },
        ]
    )

    return response

# Example request body
request_body = "This is a test email for testing@domain.com with an API token your_api_token_here"

# Sanitize the request body
sanitized_request_body = redact_request_body(request_body)

# Log to CloudWatch
log_to_cloudwatch(sanitized_request_body)

#### Closing Summary
and Common Mistakes/Follow-Ups

The above script demonstrates how to implement a basic logging filter that redacts sensitive information before it is logged. Key considerations include:

Regular Expression Customization: Ensure the regex patterns are comprehensive enough to capture all relevant tokens and PII.
Performance Impact: Be mindful of performance when processing large request bodies, especially in high-traffic environments.
False Positives/Negatives: Fine-tune the regex to minimize false positives or negatives.

#### Common mistakes
might include not thoroughly testing the regex patterns, failing to handle edge cases (e.g., nested JSON), or overlooking potential security risks if sensitive data is still processed by the script before being redacted.

For follow-ups, you could discuss:

How this solution scales with increasing volumes of logs.
Integration with other logging solutions or frameworks.
Additional layers of security and monitoring to ensure robustness.

**Code:**

```python
import boto3
import re

# Define regex patterns for identifying API tokens or PII
token_pattern = re.compile(r'\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b', re.IGNORECASE)
api_token_pattern = re.compile(r'your_api_token_here')

def redact_request_body(request_body):
    # Redact email addresses
    request_body = token_pattern.sub('<REDACTED_EMAIL>', request_body)

    # Redact API tokens
    request_body = api_token_pattern.sub('<REDACTED_TOKEN>', request_body)

    return request_body

# Function to log sanitized data to CloudWatch
def log_to_cloudwatch(sanitized_request):
    client = boto3.client('logs')

    # Define the log group and log stream name
    log_group_name = 'your-log-group-name'
    log_stream_name = f'log-stream-{sanitized_request["timestamp"]}'

    # Emit the sanitized data to CloudWatch Logs
    response = client.put_log_events(
        logGroupName=log_group_name,
        logStreamName=log_stream_name,
        logEvents=[
            {
                'timestamp': int(sanitized_request['timestamp']),
                'message': f"HTTP Request: {sanitized_request['request_body']}"
            },
        ]
    )

    return response
```

**Example input / output:**

```text
# Example request body
request_body = "This is a test email for testing@domain.com with an API token your_api_token_here"

# Sanitize the request body
sanitized_request_body = redact_request_body(request_body)

# Log to CloudWatch
log_to_cloudwatch(sanitized_request_body)
```

**Flow:**

1. Yes
2. No
3. Intercept HTTP Request
4. Parse Request Body
5. Apply Regex Patterns
6. Redact Sensitive Information
7. Emit Sanitized Data toCloudWatch
8. Log Original Data (fornon-sensitive)

