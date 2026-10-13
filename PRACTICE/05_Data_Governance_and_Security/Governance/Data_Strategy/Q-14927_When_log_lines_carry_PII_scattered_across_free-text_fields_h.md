---
type: "practice"
difficulty: "Deep"
---

# When log lines carry PII scattered across free-text fields, how do you architect a governance layer that redacts at ingestion without dropping the signals your incident models depend on?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Incident Data Reuse](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Incident_Data_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of protecting Personally Identifiable Information (PII) in log lines while preserving critical signal information, we need to architect a governance layer that can intelligently identify and redact PII fields at ingestion without compromising the essential data needed for incident models. This involves understanding the data, defining rules, implementing automated processing pipelines, and continuously monitoring and refining the system.

The core idea is to build a flexible yet robust framework that leverages natural language processing (NLP) and machine learning techniques to accurately detect PII patterns in free-text fields. We will also need to establish clear policies for what constitutes sensitive information and how it should be handled. This approach ensures data privacy while maintaining the integrity of incident analysis.

**Comparison:**

```
Pattern Type	Regex Pattern Example
Name	\b[A-Z][a-z]+\s+[A-Z][a-z]+\b
IP Address	\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b
```

**Code:**

```python
# Example Python code snippet for redacting PII
import re

def redact_pii(text, pii_patterns):
    # Define regex patterns to match PII
    pattern = re.compile('|'.join(pii_patterns))

    def replace(match):
        return '*' * len(match.group(0))

    return pattern.sub(replace, text)

# Example usage
text = "User: John Doe, IP: 192.168.1.1"
pii_patterns = [r'\b[A-Z][a-z]+\s+[A-Z][a-z]+\b', r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b']
redacted_text = redact_pii(text, pii_patterns)
print(redacted_text)  # Output: User: ***, IP: ***********
```

**Example input / output:**

```text
Input:
User: John Doe, IP: 192.168.1.1

Output (redacted):
User: ***, IP: ***********
```

**Data:**

```json
{
    "pii_patterns": [
        "\\b[A-Z][a-z]+\\s+[A-Z][a-z]+\\b",  // Name pattern
        "\\b\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\b"  // IP address pattern
    ]
}
```

**Flow:**

1. Yes
2. No
3. Ingest Log Line
4. Is PII Present?
5. Redact PII
6. PASS Through
7. Log Redacted for Storage
8. Pass to Analysis Pipeline

