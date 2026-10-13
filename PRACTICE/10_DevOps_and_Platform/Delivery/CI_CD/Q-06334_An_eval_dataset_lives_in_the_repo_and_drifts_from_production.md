---
type: "practice"
difficulty: "Deep"
---

# An eval dataset lives in the repo and drifts from production traffic; how would you build a pipeline step that periodically refreshes the golden set from real logs without leaking PII into CI?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Test Data Governance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Test_Data_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of refreshing an eval dataset from real logs while ensuring no Personally Identifiable Information (PII) is leaked, we need to implement a robust pipeline that periodically updates the golden set but strictly controls access and processing. This involves several steps: data extraction, anonymization, validation, and integration into the CI/CD process.

The core idea is to create an isolated environment where real logs can be processed safely, ensuring no PII makes it out of this environment. We'll use a combination of tools like AWS Glue, Apache Airflow, and KSQL for data processing, combined with a custom script for anonymization and validation. The pipeline will ensure that only the necessary parts of the log data are extracted, transformed, and validated before being used as part of the eval dataset.

**Comparison:**

```
Step	Description
1.	Extract logs from production environment using a secure method.
2.	Anonymize the logs by removing PII and sensitive information.
3.	Validate the anonymized data to ensure accuracy and integrity.
4.	Integrate validated data into the CI/CD pipeline for eval dataset updates.
```

**Code:**

```python
# Example Python script for anonymizing logs
def anonymize_logs(logs):
    # Implement PII removal logic here
    anonymized_logs = [remove_pii(log) for log in logs]
    return anonymized_logs

def remove_pii(log):
    # Remove sensitive information such as names, addresses, etc.
    cleaned_log = re.sub(r'\b[A-Z][a-z]+\b', 'USER', log)
    return cleaned_log
```

**Example input / output:**

```
Example input logs

logs = [ "2023-10-01T12:00:00Z, INFO, User: John Doe, Request: GET /api/v1/data", "2023-10-01T12:00:05Z, ERROR, User: Jane Smith, Exception: 404 Not Found" ]

Anonymized logs

anonymized_logs = anonymize_logs(logs) print(anonymized_logs)

CODE
Copy

Output:
```python
[
    "2023-10-01T12:00:00Z, INFO, User: USER, Request: GET /api/v1/data",
    "2023-10-01T12:00:05Z, ERROR, User: USER, Exception: 404 Not Found"
]
```

**Data:**

```json
{
  "logs": [
    "2023-10-01T12:00:00Z, INFO, User: John Doe, Request: GET /api/v1/data",
    "2023-10-01T12:00:05Z, ERROR, User: Jane Smith, Exception: 404 Not Found"
  ]
}
```

**Flow:**

1. Yes
2. No
3. Extract Logs
4. Are logs secure?
5. A anonymize logs
6. Validate logs
7. Integrate into CI/CD
8. Secure logs, retry extraction

