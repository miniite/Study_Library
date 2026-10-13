---
type: "practice"
difficulty: "Deep"
---

# What logging and observability would you add to a REST API without leaking sensitive data?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Authorization Design](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Authorization_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing logging and observability for a REST API, our primary goal is to collect sufficient information about the system's behavior while ensuring that no sensitive data is exposed. This involves carefully considering what logs we should emit, how frequently they are recorded, and which tools we use for monitoring.

At its core, this question is probing your understanding of logging best practices, observability techniques, and security principles in a fullstack SDE context. We need to ensure that our logging does not compromise user privacy or expose sensitive application details. Here’s the step-by-step approach:

Identify Key Metrics: Determine which metrics are essential for monitoring API performance and health without compromising data.
Define Logging Levels: Use appropriate log levels (DEBUG, INFO, WARNING, ERROR) to categorize messages based on their severity.
Mask Sensitive Information: Implement logging strategies that mask sensitive user or application details.
Use Secure Channels: Ensure logs are securely transmitted and stored to prevent unauthorized access.

Let's break this down from first principles:

#### Triaging the Question

The core of the question is about balancing detailed insights into API performance and health with the need to protect sensitive data. You should be able to explain how you would implement logging that captures enough information for troubleshooting while ensuring no personally identifiable information (PII) or other sensitive data are logged.

Step-by-Step Solution

Identify Key Metrics:

Performance Metrics: Response time, request size, response size.
Health Metrics: Error codes, number of requests per minute, success rate.
User Behavior Metrics: API endpoint usage patterns, frequency of specific operations.

Define Logging Levels:

INFO: Useful for understanding the flow and ensuring everything is working as expected.
PLAINTEXT
INFO: [2023-10-05 14:37:18] User requested endpoint /api/v1/users with method GET.
WARNING: Indicates a potential issue that needs attention but not an emergency.
PLAINTEXT
WARNING: [2023-10-05 14:37:19] Endpoint /api/v1/users returned slow response (1.2s).
ERROR: Indicates critical issues such as unexpected errors or exceptions.
PLAINTEXT
ERROR: [2023-10-05 14:37:20] Internal Server Error (500) while processing request to /api/v1/users.

Mask Sensitive Information:

Use placeholders or generic identifiers for sensitive data such as user IDs, email addresses, or API keys.

#### Example
of masked logging:
PLAINTEXT
INFO: [2023-10-05 14:37:18] User (UID_1234) requested endpoint /api/v1/users with method GET.

Use Secure Channels:

Ensure logs are transmitted over secure channels like TLS to prevent interception by unauthorized parties.
Use log aggregation tools that support encryption, such as ELK Stack or Splunk.

Implement Observability Tools:

Use monitoring tools like Prometheus for metrics and Grafana for visualization.
Integrate APM (Application Performance Management) tools like New Relic or Datadog to get real-time insights into API performance.
Set up alerts based on critical metrics to notify the team when issues arise.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By carefully defining what data should be logged, using appropriate log levels, masking sensitive information, and ensuring secure transmission and storage, we can maintain a robust observability system without compromising user or application security. Some common mistakes include not masking sensitive data adequately, logging too much unneeded detail, or failing to use secure channels for transmitting logs.

#### Follow-up
questions might involve discussing specific tools you prefer (e.g., ELK Stack vs. Splunk), how you handle different types of errors, and strategies for maintaining compliance with data protection regulations like GDPR.

**Comparison:**

```
Log Level	Description
DEBUG	Detailed information, typically of use only when diagnosing problems.
INFO	Confirmation that things are working as expected.
WARN	Indication of something unexpected but not necessarily an error.
ERROR	Indication of some kind of failure.
Metric Type	Example Values
Response Time	120, 2500
Error Rate	0.05%, 3%
Request Count	1000 requests/minute
```

**Code:**

```javascript
// Example of logging in a backend service
const logger = require('winston');

// INFO level log
logger.info(`User ${process.env.USER_ID} requested endpoint /api/v1/users with method GET.`);

// WARNING level log
if (responseTime > 2000) {
    logger.warn(`Endpoint /api/v1/users returned slow response (${responseTime}ms).`);
}

// ERROR level log
try {
    // Some code that might throw an error
} catch (error) {
    logger.error(`Internal Server Error (${error.message}) while processing request to /api/v1/users.`);
}
```

**Example input / output:**

```text
Input: User requested endpoint /api/v1/users with method GET.
Output: INFO: [2023-10-05 14:37:18] User UID_1234 requested endpoint /api/v1/users with method GET.

Input: Endpoint /api/v1/users returned slow response (1.2s).
Output: WARNING: [2023-10-05 14:37:19] Endpoint /api/v1/users returned slow response (1200ms).

Input: Internal Server Error (500) while processing request to /api/v1/users.
Output: ERROR: [2023-10-05 14:37:20] Internal Server Error (Error: Internal server error) while processing request to /api/v1/users.
```

**Data:**

```json
{
  "logEntry": {
    "timestamp": "2023-10-05T14:37:18Z",
    "level": "INFO",
    "message": "User UID_1234 requested endpoint /api/v1/users with method GET."
  },
  "metric": {
    "name": "responseTime",
    "value": 120
  }
}
```

**Flow:**

1. Yes
2. No
3. API Request
4. Is Endpoint Sensitive?
5. Mask PII and LogINFO/WARNING
6. Log INFO/DEBUG
7. Send Securely to Logger
8. Send Securely to Logger
9. Monitor with APM Tools
10. Monitor with APM Tools

