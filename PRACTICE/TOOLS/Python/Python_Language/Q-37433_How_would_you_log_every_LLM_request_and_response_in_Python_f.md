---
type: "practice"
difficulty: "Intermediate"
---

# How would you log every LLM request and response in Python for debugging, and what should you avoid logging for privacy reasons?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming LLM Responses](<../../../../TOOLS/Python/Python_Language/Streaming_LLM_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To log every LLM (Large Language Model) request and response in Python for debugging purposes while ensuring privacy, we need to carefully design our logging strategy. The key is to distinguish between necessary logs for debugging and sensitive information that should be avoided to protect user data. This involves implementing a structured logging approach using Python's logging module and potentially integrating with external services like a logging aggregator.

#### Triage

The question asks how to log LLM requests and responses in Python, focusing on both practical implementation and privacy considerations. The interviewer is probing your understanding of logging mechanisms, handling sensitive data, and designing robust systems that balance debugging needs with security requirements.

#### First-Principles Solution
Identify Loggable Information: Log the request parameters (e.g., user ID, timestamp) and response metadata (e.g., API status code). Avoid logging full requests or responses containing personally identifiable information (PII).
Structured Logging: Use JSON or a similar format to store logs for easy parsing.
External Aggregation: Consider using an external logging service that supports structured data storage and access controls.
Access Controls: Ensure only authorized personnel can view sensitive logs.

#### Implementation

Here's a basic implementation outline:

PYTHON
import json
import logging
from datetime import datetime

# Set up logging configuration
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def log_request_response(user_id, request_params, response):
    """Log the LLM request and response."""

    # Prepare the log entry as JSON
    log_entry = {
        "timestamp": datetime.now().isoformat(),
        "user_id": user_id,
        "request_params": request_params,
        "response_status_code": response.status_code,
        "response_content": {
            "summary": response.content[:100]  # Log a short summary to avoid PII
        }
    }

    # Convert the log entry to JSON string and log it
    logging.info(json.dumps(log_entry))

#### Closing Summary
+ Common Mistakes/Follow-Ups

This approach ensures that we are capturing enough information for debugging while protecting sensitive data. Key points include:

Avoiding Full Requests/Responses: Only log metadata or summaries.
Using JSON Format: Facilitates structured and machine-readable logs.
External Logging Services: Can provide better security controls and data storage.

#### Common mistakes
to avoid include logging full user queries, responses with PII, or failing to secure the logging mechanism properly. Always consider access controls and anonymization techniques when handling sensitive information.

#### Common Mistakes
Logging entire request bodies or response content.
Failing to sanitize logs for PII before storing.
Not securing the logging service against unauthorized access.

#### Follow-up
questions might include:

How would you handle rate limiting on log entries?
What security measures would you take if using an external logging service?

**Code:**

```python
import json
import logging
from datetime import datetime

def log_request_response(user_id, request_params, response):
    """Log the LLM request and response."""

    # Prepare the log entry as JSON
    log_entry = {
        "timestamp": datetime.now().isoformat(),
        "user_id": user_id,
        "request_params": request_params,
        "response_status_code": response.status_code,
        "response_content": {
            "summary": response.content[:100]  # Log a short summary to avoid PII
        }
    }

    # Convert the log entry to JSON string and log it
    logging.info(json.dumps(log_entry))
```

