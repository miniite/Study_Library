---
type: "practice"
difficulty: "Intermediate"
---

# What is an 'expectation' in a data validation framework and can you give an example?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Quality Fundamentals](<../../../../02_Data_Engineering/Data_Quality/Validation/Data_Quality_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a data validation framework, an expectation defines a specific condition or rule that must be met by your data to ensure its quality. These expectations are essentially checks that your data should pass before it can be considered reliable and fit for use in downstream processes. For example, if you're working with sales data, one common expectation might be that all date fields must contain valid dates.

To illustrate an expectation, let's consider a scenario where we are validating user profile data in a web application. Here’s how expectations come into play:

Expectation Definition: Define the rule or condition for the data.
Validation Process: Apply this rule to check each piece of data against predefined criteria.
Outcome Handling: Handle cases where data does not meet the expectation.

**Comparison:**

| ID | Name         | Email           | Date of Birth | Validation Status |
|----|--------------|-----------------|---------------|-------------------|
| 1  | John Doe     | johndoe@example.com | 2000-01-01    | Valid             |
| 2  | Jane Smith   | invalid_email   | 31-02-2024    | Invalid           |

**Code:**

```python
# Example expectations in Python using a hypothetical validation library

from data_validation_framework import Expectation, validate_data

def date_not_null_expectation(date_field):
    return date_field is not None

def email_format_validity_expectation(email_field):
    # Simple regex check for basic email format
    return bool(re.match(r"[^@]+@[^@]+\.[^@]+", email_field))

expectations = [
    Expectation("Date Field Should Not Be Null", date_not_null_expectation),
    Expectation("Email Format Should Be Valid", email_format_validity_expectation)
]

validated_data, validation_report = validate_data(data, expectations)
```

**Example input / output:**

```text
Input Data:
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "johndoe@example.com",
    "date_of_birth": "2000-01-01"
  },
  {
    "id": 2,
    "name": "Jane Smith",
    "email": "invalid_email",
    "date_of_birth": "31-02-2024"  // Invalid date format
  }
]

Expected Output:
{
  "valid_records": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "johndoe@example.com",
      "date_of_birth": "2000-01-01"
    }
  ],
  "invalid_records": [
    {
      "id": 2,
      "reasons": ["Email format is invalid", "Date of birth has an invalid date format"]
    }
  ]
}
```

**Data:**

```json
{
  "user_profiles": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "johndoe@example.com",
      "date_of_birth": "2000-01-01"
    },
    {
      "id": 2,
      "name": "Jane Smith",
      "email": "invalid_email",
      "date_of_birth": "31-02-2024"  // Invalid date format
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Data Valid?
5. Mark as Valid
6. Identify Validation Issues
7. Generate Validation Report
8. Store Valid Data
9. End
10. Handle Invalid Data

