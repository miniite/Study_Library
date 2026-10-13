---
type: "practice"
difficulty: "Deep"
---

# An engineer wants to log full request/response for an LLM feature to build eval sets. How do you enable that safely without hoarding PII indefinitely?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ PII Filters for LLMs](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/PII_Filters_for_LLMs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with logging requests and responses from Language Models (LLMs) in a setting where data privacy is paramount, we must ensure that personally identifiable information (PII) is handled responsibly to avoid indefinite storage or misuse. The goal here is to log enough data for evaluation sets while minimizing the risk of exposing sensitive information.

The core challenge is to strike a balance between logging necessary metadata and PII minimization. This involves understanding what PII you're dealing with, implementing proper anonymization techniques, and setting up robust retention policies.

#### Triage
Understanding the Question: The question revolves around safely logging request/response pairs for an LLM feature while avoiding the indefinite storage of Personally Identifiable Information (PII).

#### Assumptions
You are using a state-of-the-art Large Language Model.
Compliance with relevant data privacy regulations is required.
Key Concepts: PII minimization, anonymization techniques, retention policies.

#### First-Principles Solution
Identify and Anonymize PII: Understand which information in the request/response pairs constitutes PII (e.g., names, addresses, social security numbers). Use appropriate anonymization techniques to replace or mask this data.
Log Metadata Only: Log essential metadata such as timestamps, user IDs (anonymized), and interaction details without full context.
Implement Robust Retention Policies: Establish a clear policy for how long logs are retained before deletion.
Step-by-Step Solution

Analyze the Data:

Examine sample data to identify PII elements in requests and responses.
Determine which parts of the request/response need to be logged fully and which can be anonymized.

Apply Anonymization Techniques:

Use hashing or token replacement for sensitive fields like names, addresses, etc.
Mask credit card numbers using partial masking techniques (e.g., keeping the first 4 digits and last 4 digits while replacing the middle with '****').

Log Metadata Only:

Create a structured schema to log metadata such as user_id, timestamp, request_method, response_status_code.
For example, instead of logging full request/response bodies, you might only store user_id: 1234567890 and the relevant interaction details.

Set Up Retention Policies:

Define a clear retention period for logs (e.g., one year).
Implement automated systems to ensure that logs are deleted after the specified time.

#### Cons
ider using log management tools like Splunk or ELK stack to facilitate this process.

Compliance Checks and Audits:

Regularly review your logging practices to ensure compliance with data privacy regulations.
Conduct periodic audits to verify that PII is not being retained longer than necessary.

#### Closing Summary

By carefully anonymizing sensitive information, logging only essential metadata, and implementing strict retention policies, you can enable safe logging of LLM interactions without hoarding potentially harmful data. This approach ensures compliance with data privacy regulations while providing valuable data for evaluation sets.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to properly anonymize sensitive information, leading to potential breaches.
Mistake 2: Not setting up robust retention policies, resulting in indefinite storage of PII.

#### Follow-Up
Ensure regular updates and reviews of your logging strategy as data privacy laws evolve.

**Comparison:**

```
PII Type	Anonymization Technique
Name	Replace with a generic name
Address	Mask street and city, keep state
Social SSN	Replace with random numbers
```

**Code:**

```python
# Example Python code for anonymizing sensitive fields
import re

def mask_credit_card_number(cc_number):
    return f"{cc_number[:4]}****{cc_number[-4:]}"

def log_request_response(request, response):
    masked_request = {k: v if not is_pii(v) else mask_pii(v) for k, v in request.items()}
    metadata = {
        "user_id": anonymize_user_id(request['user_id']),
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
        "request_method": request.get('method'),
        "response_status_code": response.get('status_code')
    }

    # Log metadata only
    log_entry = {
        "metadata": metadata,
        "masked_request": masked_request,
        "response": {k: v if not is_pii(v) else mask_pii(v) for k, v in response.items()}
    }

    return log_entry

def is_pii(text):
    # Placeholder function to determine if text contains PII
    pass

def mask_pii(text):
    # Placeholder function to apply appropriate masking techniques
    pass

def anonymize_user_id(user_id):
    # Placeholder function for user ID anonymization
    pass
```

**Example input / output:**

```text
{
  "metadata": {
    "user_id": 1234567890,
    "timestamp": "2023-10-01 14:00:00",
    "request_method": "POST",
    "response_status_code": 200
  },
  "masked_request": {
    "user_id": 1234567890,
    "method": "POST",
    "body": {
      "name": "John Doe",
      "address": "**** ****, *** Anytown USA",
      "email": "johndoe@example.com"
    }
  },
  "response": {
    "status_code": 200,
    "message": "Success"
  }
}
```

**Data:**

```json
{
  "request": {
    "user_id": 1234567890,
    "method": "POST",
    "body": {
      "name": "John Doe",
      "address": "123 Main St, Anytown USA",
      "email": "johndoe@example.com"
    }
  },
  "response": {
    "status_code": 200,
    "message": "Success"
  }
}
```

**Flow:**

1. Yes
2. No
3. Receive Request
4. Is PII Present?
5. Apply Anonymization
6. Log Metadata Only
7. Delete Logs After RetentionPeriod
8. Log Full Data
9. Delete Logs After RetentionPeriod

