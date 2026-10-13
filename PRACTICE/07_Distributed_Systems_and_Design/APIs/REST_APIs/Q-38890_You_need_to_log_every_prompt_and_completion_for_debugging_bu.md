---
type: "practice"
difficulty: "Deep"
---

# You need to log every prompt and completion for debugging, but they contain PII and are huge. How do you design logging on the AI REST path that captures enough to debug, redacts sensitive fields, and doesn't blow up cost or latency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need a robust logging system that balances debugging needs with PII protection and cost efficiency. The core idea is to log essential metadata while redacting sensitive fields like personally identifiable information (PII) from both the prompt and completion. This involves designing a REST API endpoint for logging, implementing proper data handling in the backend, and ensuring efficient storage and retrieval mechanisms.

#### Triage

The question asks how to design logging on an AI REST path that captures enough debuggable information while redacting PII fields and managing costs and latency effectively. The key points are:

Logging Requirements: What minimal metadata is necessary for debugging?
PII Redaction: Which fields need to be redacted, and how can we ensure they are protected?
Cost Management: How do we minimize storage and processing costs without compromising on functionality?
Latency Considerations: Ensuring the logging mechanism does not introduce significant latency.

#### First-Principles Solution

Define Debuggable Metadata:

Prompt ID: Unique identifier for each prompt.
Timestamp: When the request was made.
Request Parameters: Key parameters like model version, temperature, and max tokens.
Response Code: HTTP response code indicating success or failure.
Duration: Time taken to process the request.

Redaction Strategy:

Use a tokenization approach for sensitive fields like names, addresses, etc., in both prompts and completions.
Implement regular expressions to automatically redact common PII patterns (e.g., phone numbers, email addresses).

Cost Management:

Store only the necessary metadata in logs.
Implement efficient indexing and querying mechanisms for faster retrieval of relevant data.
Use cost-effective storage solutions like S3 or a NoSQL database.

Latency Considerations:

Ensure that logging is done asynchronously to avoid blocking the main request flow.
Optimize the backend to handle high volumes of log entries without affecting performance.
Detailed Implementation

API Endpoint for Logging:

HTTP
POST /api/log/prompt_completion

Request Body:

JSON
{
  "prompt_id": "unique_id_123",
  "timestamp": "2023-10-05T14:48:00Z",
  "request_params": {
    "model_version": "v2.1",
    "temperature": 0.7,
    "max_tokens": 100
  },
  "response_code": 200,
  "duration": 0.5
}

Backend Implementation:

Use a microservice architecture to handle logging.
Implement an asynchronous worker that processes logs in the background.

Data Storage and Indexing:

PYTHON
import json
from elasticsearch import Elasticsearch

def log_prompt_completion(prompt_id, timestamp, request_params, response_code, duration):
    es = Elasticsearch([{'host': 'localhost', 'port': 9200}])
    document = {
        "prompt_id": prompt_id,
        "timestamp": timestamp,
        "request_params": json.dumps(request_params),
        "response_code": response_code,
        "duration": duration
    }
    es.index(index="log_index", body=document)

Redaction Logic:

PYTHON
import re

def redact_pii(text):
    pii_patterns = [
        r"\b\d{3}[-\.\s]?\d{2}[-\.\s]?\d{4}\b",  # Phone numbers
        r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b"  # Emails
    ]
    for pattern in pii_patterns:
        text = re.sub(pattern, "REDACTED", text)
    return text

def log_prompt(prompt):
    prompt_text = redact_pii(prompt)
    # Log the redacted prompt
    log_prompt_completion(prompt_id="unique_id_123", timestamp="now", request_params={"prompt": prompt_text}, response_code=200, duration=0.5)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By logging essential metadata while redacting sensitive fields, using efficient storage mechanisms, and ensuring asynchronous processing, we can meet the logging requirements without blowing up costs or latency.

#### Common Mistakes
Failing to define clear metadata needs, leading to unnecessary data capture.
Not implementing robust redaction logic, exposing PII in logs.
Ignoring cost considerations, leading to expensive storage and retrieval.

#### Follow-Ups
How would you handle logging errors or failures during the logging process?
What measures would you take to ensure compliance with data protection regulations like GDPR?

**Comparison:**

```
Field	Type	Description
prompt_id	String	Unique identifier for each prompt
timestamp	DateTime	When the request was made
request_params	JSON	Key parameters like model version, temperature, and max tokens
response_code	Integer	HTTP response code indicating success or failure
duration	Float	Time taken to process the request
```

**Code:**

```python
import json
from elasticsearch import Elasticsearch

def log_prompt_completion(prompt_id, timestamp, request_params, response_code, duration):
    es = Elasticsearch([{'host': 'localhost', 'port': 9200}])
    document = {
        "prompt_id": prompt_id,
        "timestamp": timestamp,
        "request_params": json.dumps(request_params),
        "response_code": response_code,
        "duration": duration
    }
    es.index(index="log_index", body=document)

def redact_pii(text):
    pii_patterns = [
        r"\b\d{3}[-\.\s]?\d{2}[-\.\s]?\d{4}\b",  # Phone numbers
        r"\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b"  # Emails
    ]
    for pattern in pii_patterns:
        text = re.sub(pattern, "REDACTED", text)
    return text

def log_prompt(prompt):
    prompt_text = redact_pii(prompt)
    log_prompt_completion(prompt_id="unique_id_123", timestamp="now", request_params={"prompt": prompt_text}, response_code=200, duration=0.5)
```

**Example input / output:**

```text
POST /api/log/prompt_completion
Content-Type: application/json

{
  "prompt_id": "unique_id_123",
  "timestamp": "2023-10-05T14:48:00Z",
  "request_params": {
    "model_version": "v2.1",
    "temperature": 0.7,
    "max_tokens": 100
  },
  "response_code": 200,
  "duration": 0.5
}
```

**Data:**

```json
{
  "prompt_id": "unique_id_123",
  "timestamp": "2023-10-05T14:48:00Z",
  "request_params": {
    "model_version": "v2.1",
    "temperature": 0.7,
    "max_tokens": 100
  },
  "response_code": 200,
  "duration": 0.5
}
```

**Flow:**

1. Yes
2. No
3. Receive Prompt/Completion
4. Is PII Present?
5. Redact PII
6. Log Metadata
7. Store Log in Elasticsearch

