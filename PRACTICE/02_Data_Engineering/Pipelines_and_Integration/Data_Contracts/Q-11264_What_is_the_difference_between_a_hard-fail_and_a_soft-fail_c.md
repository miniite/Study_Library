---
type: "practice"
difficulty: "Deep"
---

# What is the difference between a hard-fail and a soft-fail contract violation and when do you use each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Contract Practice](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data engineering, ensuring that our datasets adhere to predefined rules or contracts is crucial. A data contract defines the expected structure, format, and integrity of the data. The distinction between hard-fail and soft-fail violations in these contracts is fundamental for managing data quality and maintaining system stability.

A hard-fail violation occurs when a dataset violates a critical data contract that cannot be ignored or bypassed by the system. This typically results in the rejection of the entire dataset, forcing the source to correct the issue before reprocessing. Hard failures are used when the violation is so severe it compromises the integrity or accuracy of the data.

A soft-fail violation, on the other hand, allows for a more lenient approach where only the problematic parts of the data can be rejected or corrected while allowing the rest to proceed. This ensures that the system remains operational and does not get halted due to minor issues in the input data.

In summary, hard-fails are used when critical rules must be strictly enforced, whereas soft-fails offer a more flexible approach for handling less severe violations without halting the entire process.

**Comparison:**

| Violation Type | Description                          | Impact                     |
|----------------|--------------------------------------|----------------------------|
| Hard-Fail      | Critical rule violation              | Dataset rejection          |
| Soft-Fail      | Minor or non-critical issue          | Partial data rejection     |

**Code:**

```python
# Example of a hard-fail condition in Python
def validate_data(data):
    if not data['id']:
        raise ValueError("ID field cannot be empty")
    return True

# Example of a soft-fail condition with error logging
def validate_data_soft(data, log_errors=False):
    errors = []
    if not data['id']:
        errors.append("ID field is missing or empty")
    if len(errors) > 0:
        for err in errors:
            print(err)
        return False
    return True
```

**Example input / output:**

```

Hard-Fail Example:

Input:

JSON
Copy
{
  "id": "",
  "name": "John Doe",
  "age": 30,
  "email": "invalid_email"
}

Output:

CODE
Copy
ValueError: ID field cannot be empty

The entire dataset is rejected due to the hard-fail condition on the id field.

Soft-Fail Example:

Input:

JSON
Copy
{
  "id": "",
  "name": "John Doe",
  "age": 30,
  "email": "invalid_email"
}

Output:

CODE
Copy
ID field is missing or empty

{'name': 'John Doe', 'age': 30, 'email': 'invalid_email'}

The id field's error is logged and the rest of the data proceeds.
```

**Data:**

```json
{
  "data": {
    "id": "",
    "name": "John Doe",
    "age": 30,
    "email": "invalid_email"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Contract Validation
4. Is Violation Critical?
5. Hard-Fail: Reject Data
6. Soft-Fail: Log Error & Proceed
7. Notify Source to Correct
8. Proceed with Clean Data

