---
type: "practice"
difficulty: "Deep"
---

# Explain how you would automate PII discovery/classification across thousands of columns. What's the false-positive risk and cost?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Glossary Metadata and Contracts](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To automate PII (Personally Identifiable Information) discovery and classification across thousands of columns, we need a robust approach that balances accuracy with performance. The goal is to identify sensitive data without compromising system performance or incurring excessive costs. This involves several steps:

Understand the Requirements: First, define what constitutes PII for your organization. Common types include names, addresses, phone numbers, and social security numbers. Understanding these definitions will guide our detection logic.
Choose a Classification Model: We'll use a combination of static rules (e.g., regex patterns) and machine learning models to classify data.
Implement Automated Discovery: The solution should be scalable, efficient, and able to handle large datasets without overloading the system.

The key steps are:

Data Collection & Preprocessing:
Collect metadata about tables, columns, and their relationships.
Use a schema to understand the data structure.
Rule-Based Detection: Implement regex patterns for common PII types.
Machine Learning Classification: Train models using historical or synthetic datasets to improve accuracy.
Real-Time vs Batch Processing: Decide between real-time processing for critical applications and batch processing for less time-sensitive use cases.

False-positive risk refers to the chances that data will be incorrectly classified as PII, which can lead to unnecessary data handling or storage restrictions. The cost includes both false positives (unnecessary restrictions) and false negatives (missed PII).

**Comparison:**

| Column Name | Data Type | PIIs Classified |
|-------------|----------|----------------|
| first_name  | text     | true           |
| last_name   | text     | true           |
| phone       | text     | true           |
| address     | text     | false          |

**Code:**

```python
import re

# Define regex patterns for common PII types
pii_patterns = {
    'name': r'\b[A-Z][a-z]+\b',
    'phone': r'[0-9]{3}-[0-9]{3}-[0-9]{4}',
    # Add more patterns as needed
}

def detect_pii(text, pii_patterns):
    for key, pattern in pii_patterns.items():
        if re.search(pattern, text):
            return key
    return None

# Example usage
text = "John Doe lives at 123 Elm St and can be reached at 555-123-4567"
detected_pii = detect_pii(text, pii_patterns)
print(detected_pii)  # Output: 'name', 'phone'
```

**Example input / output:**

```text
Input:
- Table name: 'users', column name: 'first_name'

Output:
- PII classification result for 'first_name' in 'users': true

Dry-run:
1. Metadata collected from 'users' table.
2. Pattern 'name' applied to 'first_name'.
3. 'first_name' matched regex pattern, hence classified as PII.
```

**Data:**

```json
{
    "schema": {
        "table_name": "string",
        "column_name": "string",
        "data_type": "string",
        "pii_classification": "boolean"
    },
    "example_rows": [
        {"table_name": "users", "column_name": "first_name", "data_type": "text", "pii_classification": true},
        {"table_name": "addresses", "column_name": "street_address", "data_type": "text", "pii_classification": false}
    ]
}
```

**Flow:**

1. Static Rules
2. ML Model
3. Start
4. Collect Metadata
5. Static Rules or ML Model?
6. Detect PII using regex
7. Train and Classify with ML
8. Post-processing, e.g., falsepositive filtering
9. Store Results

