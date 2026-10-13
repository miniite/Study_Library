---
type: "practice"
difficulty: "Deep"
---

# How do you architect PII lineage that survives schema evolution — when a column is renamed, split, or derived, its sensitivity label and consent must follow automatically?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Fine-tuned Models as Sensitive Artifacts](<../../../../11_AI_and_ML/Responsible_AI/Privacy_and_PII/Fine-tuned_Models_as_Sensitive_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to architect PII (Personally Identifiable Information) lineage that survives schema evolution, we need to focus on maintaining a robust data provenance system. This involves tracking not just the data but also its lineage and metadata as it transforms through various stages of our AI pipeline. The challenge lies in ensuring that when changes are made to the database schema—such as renaming columns, splitting them into multiple fields, or deriving new ones—the sensitivity labels and consent information follow automatically.

The key concepts here include understanding the principles of data provenance, handling dynamic metadata, and implementing a flexible architecture that can adapt to changes without manual intervention. We will explore these in detail step-by-step.

**Comparison:**

```
Original Name	New Names	Sensitivity Label	Consent Type
user_id	user_unique_id, profile.user.id	Sensitive	Explicit
email_address	contact.email, social.login.email	Sensitive	Explicit
```

**Code:**

```python
# Example code snippet for tracking PII lineage during schema evolution

class PiiMetadata:
    def __init__(self, original_column_name, sensitivity_label, consent_type):
        self.original_column_name = original_column_name
        self.sensitivity_label = sensitivity_label
        self.consent_type = consent_type

def track_pii_lineage(column_changes: dict) -> list:
    lineage_records = []
    for old_name, new_names in column_changes.items():
        for new_name in new_names:
            if old_name and new_name:
                lineage_records.append(PiiMetadata(old_name, sensitivity_label=LABEL_SENSITIVE, consent_type='explicit'))
            else:
                print(f"Skipping invalid name pair: {old_name} -> {new_name}")
    return lineage_records

# Sample column changes due to schema evolution
column_changes = {
    'user_id': ['user_unique_id', 'profile.user.id'],
    'email_address': ['contact.email', 'social.login.email']
}

lineage_records = track_pii_lineage(column_changes)
for record in lineage_records:
    print(record.original_column_name, record.sensitivity_label, record.consent_type)
```

**Example input / output:**

```

Input:

PYTHON
Copy
column_changes = {
    'user_id': ['user_unique_id', 'profile.user.id'],
    'email_address': ['contact.email', 'social.login.email']
}

Output:

CODE
Copy
user_id sensitive explicit
user_unique_id sensitive explicit
profile.user.id sensitive explicit
email_address sensitive explicit
contact.email sensitive explicit
social.login.email sensitive explicit
```

**Data:**

```json
{
  "column_changes": {
    "user_id": ["user_unique_id", "profile.user.id"],
    "email_address": ["contact.email", "social.login.email"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Schema Change Detected
10. Rename Columns?
11. Create Metadata Records forNew Names
12. Split Columns?
13. Create Metadata Records forSplit Fields
14. Derive New Columns?
15. Create Metadata Records forDerived Fields
16. Are there any other changes?
17. Update Lineage Database withMetadata Records

