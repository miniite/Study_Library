---
type: "practice"
difficulty: "Deep"
---

# How would you set up an automated PII discovery and classification scan across a large heterogeneous estate and manage the false positives it produces?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Preventing PII Leakage](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Preventing_PII_Leakage.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up an automated PII (Personally Identifiable Information) discovery and classification scan in a large, heterogeneous database estate, we need to focus on creating a comprehensive system that can handle diverse data sources, classify data accurately, and manage false positives effectively. This involves several key steps from first principles.

Firstly, let's understand the core requirements: identifying PII, classifying it correctly, and managing false positives. We will leverage existing tools and build custom scripts to ensure robustness and scalability.

**Comparison:**

```
Column	PII Type	Severity
SSN	Social Security Number	High
Date of Birth	Personal Identifier	Medium
Home Address	Location Information	High
```

**Code:**

```python
# Example Python script for scanning data
import pandas as pd

def scan_pii(dataframe):
    pii_columns = detect_pii_columns()
    results = []
    for col in pii_columns:
        # Implement logic to extract potential PII
        results.append((col, get_pii_from_column(col)))
    return results

def detect_pii_columns():
    # Placeholder function to identify likely PII columns based on metadata or heuristic analysis
    pass

def get_pii_from_column(column):
    # Placeholder function to extract and classify PII from a column
    pass

# Example usage
data = pd.read_csv('database_dump.csv')
results = scan_pii(data)
print(results)
```

**Example input / output:**

| Column | Detected PII | Classification |
|--------|--------------|----------------|
| SSN    | Yes          | High           |
| DOB    | Yes          | Medium         |
| Home   | No           | None           |

**Data:**

```json
{
  "pii_columns": ["SSN", "Date of Birth", "Home Address"],
  "classification_rules": {
    "SSN": "High",
    "Date of Birth": "Medium",
    "Home Address": "High"
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Start
5. Is data ready for scanning?
6. Detect PII columns
7. Classify detected PII
8. Are false positivesmanageable?
9. Refine classification rules
10. Retest and adjust
11. End

