---
type: "practice"
difficulty: "Basic"
---

# What does 'garbage in garbage out' mean in the context of data pipelines?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Quality Fundamentals](<../../../../02_Data_Engineering/Data_Quality/Validation/Data_Quality_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the realm of data engineering, the phrase "garbage in, garbage out" (GIGO) encapsulates a fundamental principle: the quality and accuracy of outputs from any system or process directly depend on the quality of its inputs. In the context of data pipelines, GIGO means that if your source data is flawed, corrupted, or missing key information, then no amount of subsequent processing will make up for these initial flaws. This concept underscores the critical importance of robust data validation and cleansing processes at every stage of the pipeline.

**Code:**

```python
# Example Python code to illustrate basic data validation
def validate_data(row):
    if not row['customer_id'] or len(row['customer_id']) < 5:
        return False
    if not isinstance(row['amount'], (int, float)) or row['amount'] <= 0:
        return False
    # Additional checks can be added here
    return True

# Sample data to validate
sample_data = [
    {'customer_id': '12345', 'amount': 99.99},
    {'customer_id': '', 'amount': -99.99},  # Invalid: Empty customer_id and negative amount
    {'customer_id': '54321', 'amount': 0}   # Valid, but potentially problematic
]

# Validate the data
valid_data = [row for row in sample_data if validate_data(row)]
print("Valid Data:", valid_data)
```

**Example input / output:**

```

Input:

Sample data with various issues (empty customer_id, invalid amount value).

Output:

Validated list of data, where the second entry is discarded as it violates validation rules.
CODE
Copy
Valid Data: [{'customer_id': '12345', 'amount': 99.99}]
```

**Data:**

```json
[
    {"customer_id": "12345", "amount": 99.99},
    {"customer_id": "", "amount": -99.99},  // Invalid: Empty customer_id and negative amount
    {"customer_id": "54321", "amount": 0}   // Valid, but potentially problematic
]
```

