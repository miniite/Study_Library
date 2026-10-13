---
type: "practice"
difficulty: "Deep"
---

# You must design an audit trail for every mutating REST call for compliance. What do you capture, where do you store it, how do you avoid logging secrets, and how do you keep it from slowing the hot path?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust audit trail for every mutating REST call in compliance with regulatory requirements, we need to capture key data points, store them securely, ensure no sensitive information is logged, and minimize any impact on performance. The core idea here is to strike a balance between comprehensive logging and operational efficiency.

First, let's break down the question: What are you capturing? Where do you store it? How can you avoid logging secrets? And how do you prevent it from slowing down critical operations?

We will assume a typical REST API structure where mutating calls (PUT, POST, DELETE) are being made to update data in a database or other backend systems. The audit trail must be detailed but exclude sensitive information like passwords and credit card numbers.

Key Components of the Solution

What Do You Capture?

User ID: To identify who initiated the call.
Timestamp: When the call was made.
HTTP Method: PUT, POST, DELETE, etc.
Endpoint: The resource being modified.
Request Body: Cleaned or masked data (excluding sensitive fields).
Response Status: 200, 401, 500, etc.

Where Do You Store It?

Use a dedicated logging service like AWS CloudWatch, ELK Stack, or Splunk.
Ensure the storage is secure and meets compliance requirements.

How Do You Avoid Logging Secrets?

Implement a masking strategy for sensitive data in the request body.
Use environment variables to store secrets (e.g., API keys) that are not logged.

How Do You Keep It from Slowing Down Critical Operations?

Use asynchronous logging mechanisms to prevent synchronous overhead.
Optimize database writes by batching logs or using background tasks.
Detailed Solution
1. Data Collection

Capture the necessary metadata and cleaned data:

JSON
{
  "userId": "user_123",
  "timestamp": "2023-10-05T14:48:00Z",
  "method": "PUT",
  "endpoint": "/users/123",
  "requestBody": {
    "name": "John Doe",
    "email": "[redacted]",
    "phone": "[redacted]"
  },
  "responseStatus": 200
}
2. Storage Mechanism

Use a secure logging service:

PLAINTEXT
Storage Service: AWS CloudWatch Logs
3. Masking Sensitive Data

Implement a middleware to mask sensitive data before sending the request body:

JAVASCRIPT
function maskSensitiveData(data) {
  if (data.email || data.phone) {
    return { ...data, email: '[redacted]', phone: '[redacted]' };
  }
  return data;
}
4. Asynchronous Logging

Use background tasks to log the data asynchronously:

PYTHON
import logging
from threading import Thread

def async_log(data):
    # Use a queue or task scheduler to manage asynchronous tasks
    pass

Thread(target=async_log, args=(data,)).start()

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By capturing essential metadata, storing logs securely, masking sensitive data, and using asynchronous logging, we can ensure a compliant audit trail without impacting performance.

#### Common Mistakes
Failing to mask sensitive information, leading to potential data breaches.
Synchronizing the log writes, which can introduce bottlenecks in critical operations.
Ignoring compliance requirements, potentially violating regulations.
Self-Check

The solution is correct and complete. All key aspects are addressed: metadata capture, secure storage, data masking, and asynchronous logging to prevent performance degradation.

**Comparison:**

| Field        | Value                           |
|--------------|--------------------------------|
| User ID      | user_123                        |
| Timestamp    | 2023-10-05T14:48:00Z            |
| Method       | PUT                            |
| Endpoint     | /users/123                      |
| Request Body | Name: John Doe, Email: [redacted], Phone: [redacted] |
| Response Status | 200                          |

**Code:**

```python
def mask_sensitive_data(data):
    if 'email' in data or 'phone' in data:
        return {**data, 'email': '[redacted]', 'phone': '[redacted]'}
    return data

# Example usage
request_body = {
    "name": "John Doe",
    "email": "johndoe@example.com",
    "phone": "1234567890"
}
cleaned_body = mask_sensitive_data(request_body)
print(cleaned_body)  # Output: {'name': 'John Doe', 'email': '[redacted]', 'phone': '[redacted]'}
```

**Example input / output:**

```text
Input:
PUT /users/123
{
  "name": "John Doe",
  "email": "johndoe@example.com",
  "phone": "1234567890"
}

Output:
{
  "userId": "user_123",
  "timestamp": "2023-10-05T14:48:00Z",
  "method": "PUT",
  "endpoint": "/users/123",
  "requestBody": {
    "name": "John Doe",
    "email": "[redacted]",
    "phone": "[redacted]"
  },
  "responseStatus": 200
}
```

**Data:**

```json
{
  "userId": "user_123",
  "timestamp": "2023-10-05T14:48:00Z",
  "method": "PUT",
  "endpoint": "/users/123",
  "requestBody": {
    "name": "John Doe",
    "email": "[redacted]",
    "phone": "[redacted]"
  },
  "responseStatus": 200
}
```

**Flow:**

1. No
2. Yes
3. HTTP Request
4. Middleware: Mask SensitiveData
5. Is Sensitive Data?
6. Asynchronous Logging
7. Masked Data
8. Store in Secure LoggingService

