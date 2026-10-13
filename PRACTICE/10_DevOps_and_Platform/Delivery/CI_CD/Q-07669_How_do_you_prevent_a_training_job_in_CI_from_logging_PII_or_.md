---
type: "practice"
difficulty: "Deep"
---

# How do you prevent a training job in CI from logging PII or raw sensitive features into experiment-tracking artifacts?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Performance Regression Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Performance_Regression_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In this scenario, we need to ensure that our Continuous Integration (CI) pipelines do not inadvertently log Personally Identifiable Information (PII) or other sensitive data like raw feature vectors during machine learning model training. This involves implementing a series of strategies and tools at the pipeline level to filter out sensitive information before it gets logged.

#### Triaging the Question

The core question is about preventing PII or sensitive features from being logged into experiment-tracking artifacts in a CI environment. The interviewer may be probing your understanding of data privacy, security best practices, and how these can be integrated into CI/CD pipelines. Key assumptions include that we are using some form of logging for tracking experiments, and that the training jobs are part of an automated CI pipeline.

Step-by-Step Solution:

Data Masking:

Implement data masking techniques to replace or obfuscate sensitive values in logs. For example, use tools like logstash with plugins that can mask PII fields.

Feature Engineering Pipelines:

Ensure that the feature engineering pipeline does not retain raw, sensitive features longer than necessary. Store only anonymized and aggregated data for experimentation purposes.

Logging Configuration:

Adjust logging configurations to exclude specific columns or entries related to sensitive data. Use environment variables or configuration files to define which fields should be logged.

Regular Audits:

Perform regular audits of the CI logs to ensure compliance with privacy policies and regulations like GDPR or HIPAA.

Security Policies:

Implement strict security policies for access to logging systems, ensuring that only authorized personnel can view sensitive data in logs.

Using Secure Logging Services:

Utilize secure logging services that provide features such as encryption at rest and in transit, and automatic redaction of sensitive information.

#### Example
Implementation:

Let's walk through a basic example using logstash for log masking:

BASH
# Logstash Configuration File
input {
  stdin { }
}

filter {
  if [message] =~ /pii/ {
    mutate {
      add_field => { "[masked_message]" => "%{[message]}_MASKED" }
      remove_field => ["message"]
    }
  }
}

output {
  stdout { codec => rubydebug }
}

In this example, any log message containing the word "pii" is replaced with a masked version.

#### Closing Summary

To prevent PII or raw sensitive features from being logged in CI/CD pipelines, we need to implement a combination of data masking techniques, secure logging configurations, regular audits, and strict security policies. By carefully managing how sensitive information is handled and stored, we can ensure that experiments are tracked without compromising privacy.

#### Common Mistakes/Follow-Ups
Not Masking Data Properly: Failing to use tools like logstash or equivalent for masking sensitive data.
Inadequate Security Policies: Not implementing strict access controls on logging systems.
Failure to Regularly Audit Logs: Missing out on periodic checks that ensure compliance with privacy policies.

**Comparison:**

| Log Entry          | Masked Entry       |
|--------------------|-------------------|
| User John Doe      | User [MASKED]      |
| ID 123             | ID [MASKED]        |
| Logged in          | Logged in         |

**Code:**

```
RUBY
Copy
# Example Logstash Filter Configuration
input {
  stdin { }
}

filter {
  if [message] =~ /pii/ {
    mutate {
      add_field => { "[masked_message]" => "%{[message]}_MASKED" }
      remove_field => ["message"]
    }
  }
}

output {
  stdout { codec => rubydebug }
}
```

**Example input / output:**

```text
Input:
{
  "timestamp": "2023-10-15T14:00:00Z",
  "message": "User John Doe with ID 123 logged in",
  "level": "INFO"
}

Output after masking sensitive data:
{
  "timestamp": "2023-10-15T14:00:00Z",
  "masked_message": "User [MASKED] with ID [MASKED] logged in",
  "level": "INFO"
}
```

**Data:**

```json
{
  "log_entry": {
    "timestamp": "2023-10-15T14:00:00Z",
    "message": "User John Doe with ID 123 logged in",
    "level": "INFO"
  }
}
```

**Flow:**

1. Yes
2. No
3. Log Entry
4. Is PII?
5. Mask Data
6. Send to Log Service

