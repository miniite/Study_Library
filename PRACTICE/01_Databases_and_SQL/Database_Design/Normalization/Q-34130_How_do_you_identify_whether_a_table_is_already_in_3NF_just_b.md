---
type: "practice"
difficulty: "Intermediate"
---

# How do you identify whether a table is already in 3NF just by looking at it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine if a table is in Third Normal Form (3NF), we need to check that it satisfies two main conditions:

The table is in Second Normal Form (2NF).
Every non-key attribute is functionally dependent on the primary key.

Given these criteria, let's break down the process step by step using first principles and reason through common scenarios.

**Code:**

```python
# Example of a simple table schema for illustration
customer = {
    "id": 1,
    "name": "Alice",
    "address": "123 Main St, Cityville",
    "city_id": 5,
    "phone_number": "9876-5432"
}
```

**Example input / output:**

```text
graph LR;
    A[Customer Table] --> B{Is Primary Key Unique?};
    B -- Yes --> C[Check for Transitive Dependencies];
    C --> D[Does Every Non-Key Attribute Depend Only on PK?]
    B -- No --> E[Table is not in 2NF, no need to check further]

    A --> F{Are All Non-Keys Functionally Dependent on Primary Key?}
    F -- Yes --> G[Table is in 3NF];
    F -- No --> H[Table has partial dependencies; needs redesign]
```

**Data:**

```json
{
  "customers": [
    {
      "id": 1,
      "name": "Alice",
      "address": "123 Main St, Cityville",
      "city_id": 5,
      "phone_number": "9876-5432"
    },
    {
      "id": 2,
      "name": "Bob",
      "address": "456 Elm St, Townsville",
      "city_id": 6,
      "phone_number": "1234-5678"
    }
  ]
}
```

