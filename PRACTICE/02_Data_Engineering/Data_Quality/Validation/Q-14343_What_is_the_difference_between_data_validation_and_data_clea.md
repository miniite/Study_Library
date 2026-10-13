---
type: "practice"
difficulty: "Basic"
---

# What is the difference between data validation and data cleansing?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Validation Coverage and Quarantine](<../../../../02_Data_Engineering/Data_Quality/Validation/Validation_Coverage_and_Quarantine.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Data validation and data cleansing are two critical steps in ensuring data quality, but they serve different purposes. Data validation checks if the data meets specific criteria or rules, while data cleansing focuses on fixing errors and inconsistencies in the existing data.

#### Triaging the Question

The interviewer is likely probing your understanding of the fundamental concepts behind data quality management. They want to know whether you can distinguish between these two important steps and understand their roles in maintaining data integrity.

#### First-Principles Solution
Data Validation

Data validation involves checking that data conforms to predefined rules or standards before it is stored or processed. For example, if a customer's age must be a positive number, a validation rule would ensure this condition is met during the input process. Here are some key aspects of data validation:

Rule Definition: Define clear and specific criteria (e.g., age > 0).
Real-time Checks: Validate data as it is entered to prevent incorrect entries.
Immediate Feedback: Provide immediate feedback to users if data does not meet the rules.
Data Cleansing

Data cleansing, on the other hand, involves correcting or removing errors and inconsistencies from existing datasets after they have been collected. This process aims to improve the overall quality of the data by making it more accurate and consistent. Common tasks include:

Duplicate Removal: Identifying and eliminating duplicate records.
Missing Value Handling: Filling in missing values appropriately (e.g., using imputation methods).
Data Typing: Ensuring that data types are correctly applied (e.g., converting "05-08-19" to a date format).
Outlier Removal: Identifying and handling outliers that may skew the dataset.

#### Closing Summary

In summary, validation focuses on preventing bad data from entering the system, while cleansing deals with fixing issues in already collected data. Both are crucial for maintaining high-quality datasets.

#### Common Mistakes/Follow-Ups
Confusion Between Validation and Cleansing: It's common to mistakenly treat these as interchangeable processes.
Ignoring Validation Rules During Data Collection: Ensuring that validation rules are robust is critical, but they should not be bypassed during initial data entry.
Overlooking the Importance of Both Steps: Focusing only on one aspect can lead to suboptimal results. A balanced approach involving both validation and cleansing is necessary.

#### Example Scenario

#### Cons
ider a user uploading customer details to a system. The validation would check that all mandatory fields are filled, ages are positive integers, and emails have valid formats. If any of these criteria fail, the system would return an error message and not proceed with processing the data. Post-upload, if some records contain duplicated email addresses or incorrect phone numbers, the cleansing process would be triggered to resolve these issues.

**Comparison:**

```
Rule	Action
Empty Email	Reject data or request new input
Negative Age	Log error and correct
```

**Code:**

```python
# Example validation function in Python
def validate_customer_data(data):
    required_fields = ["name", "email", "age"]
    errors = []

    for field in required_fields:
        if not data.get(field, None):
            errors.append(f"Missing required field: {field}")

    if data["age"] < 0:
        errors.append("Age must be a positive integer")

    return errors
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "customer": {
    "name": "John Doe",
    "email": "",
    "age": -25,
    "phone": "123-456-7890"
  }
}

Output:

PLAINTEXT
Copy
[Missing required field: 'email', Age must be a positive integer]
```

**Data:**

```json
{
  "customer": {
    "name": "John Doe",
    "email": "",
    "age": -25,
    "phone": "123-456-7890"
  }
}
```

**Flow:**

1. Yes
2. No
3. Data Entry
4. Validation Check
5. Errors?
6. Cleansing Step
7. Proceed to Processing

