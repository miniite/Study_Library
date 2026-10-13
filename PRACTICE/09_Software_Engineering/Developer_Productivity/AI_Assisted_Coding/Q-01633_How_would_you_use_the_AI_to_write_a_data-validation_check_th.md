---
type: "practice"
difficulty: "Intermediate"
---

# How would you use the AI to write a data-validation check that fails loudly if a column has an unexpected range or dtype?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Reviewing AI Code for Correctness Bugs](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Reviewing_AI_Code_for_Correctness_Bugs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we'll create a robust data-validation function using an AI tool like Copilot/Cursor. This function will ensure that columns in a DataFrame have expected ranges and dtypes by performing type checks and range validations. Here's a step-by-step explanation:

#### Overview
The core idea is to define a function that iterates through each column of the DataFrame, checks its dtype and value range, and raises an informative exception if any column does not meet the specified criteria.

#### Triage
The question asks for an implementation that “fails loudly,” meaning it should clearly indicate what went wrong when validation fails. We need to define expected dtypes and valid ranges, handle different data types (e.g., integer, float), and provide detailed error messages.

#### First-Principles Solution
Define the expected range for each column based on domain knowledge.
Use Python's pandas library to manipulate and validate the DataFrame.
Implement a function that checks both dtype and value ranges.

Here’s how we can implement this:

PYTHON
import pandas as pd

def validate_dataframe(df: pd.DataFrame, expected_dtypes: dict, valid_ranges: dict) -> None:
    """
    Validate each column in the DataFrame against expected dtypes and value ranges.

    :param df: Input DataFrame to be validated.
    :param expected_dtypes: Dictionary mapping column names to expected dtypes (e.g., int, float).
    :param valid_ranges: Dictionary mapping column names to their valid value ranges (inclusive or exclusive).
    """
    for col in df.columns:
        # Check dtype
        if col not in expected_dtypes or not isinstance(df[col].dtype, expected_dtypes[col]):
            raise ValueError(f"Unexpected dtype '{df[col].dtype}' for column {col}. Expected: {expected_dtypes[col]}")

        # Check value range (if applicable)
        if col in valid_ranges:
            min_val, max_val = valid_ranges[col]
            if not all(min_val <= val <= max_val for val in df[col]):
                raise ValueError(f"Column '{col}' has values outside the expected range [{min_val}, {max_val}].")

# Example usage
data = {
    'age': [18, 25, 30, 45],
    'salary': [30000, 60000, 90000, 120000]
}
df = pd.DataFrame(data)

expected_dtypes = {'age': int, 'salary': float}
valid_ranges = {
    'age': (18, 100),
    'salary': (15000, 300000)
}

try:
    validate_dataframe(df, expected_dtypes, valid_ranges)
except ValueError as e:
    print(e)

# Expected output: No exception is raised because the data is within the expected ranges.

#### Closing Summary
This approach ensures that any unexpected dtypes or value ranges are caught and reported clearly. The function validate_dataframe can be easily extended to include more complex validation logic if needed.

#### Common Mistakes/Follow-Ups

Not handling all possible data types correctly.
Failing to validate the entire range of values, which might lead to silent failures.
Not considering edge cases like NaNs or infinite values in certain dtypes.

**Comparison:**

| Column | Expected DType | Valid Range |
|--------|---------------|-------------|
| age    | int           | 18-100      |
| salary | float         | 15000-300000|

**Code:**

```python
import pandas as pd

def validate_dataframe(df: pd.DataFrame, expected_dtypes: dict, valid_ranges: dict) -> None:
    """
    Validate each column in the DataFrame against expected dtypes and value ranges.

    :param df: Input DataFrame to be validated.
    :param expected_dtypes: Dictionary mapping column names to expected dtypes (e.g., int, float).
    :param valid_ranges: Dictionary mapping column names to their valid value ranges (inclusive or exclusive).
    """
    for col in df.columns:
        # Check dtype
        if col not in expected_dtypes or not isinstance(df[col].dtype, expected_dtypes[col]):
            raise ValueError(f"Unexpected dtype '{df[col].dtype}' for column {col}. Expected: {expected_dtypes[col]}")

        # Check value range (if applicable)
        if col in valid_ranges:
            min_val, max_val = valid_ranges[col]
            if not all(min_val <= val <= max_val for val in df[col]):
                raise ValueError(f"Column '{col}' has values outside the expected range [{min_val}, {max_val}].")
```

**Example input / output:**

```text
data = {
    'age': [18, 25, 30, 45],
    'salary': [30000, 60000, 90000, 120000]
}
df = pd.DataFrame(data)

expected_dtypes = {'age': int, 'salary': float}
valid_ranges = {
    'age': (18, 100),
    'salary': (15000, 300000)
}

try:
    validate_dataframe(df, expected_dtypes, valid_ranges)
except ValueError as e:
    print(e)

# Output: No exception is raised because the data is within the expected ranges.
```

**Data:**

```json
{
    "data": {
        "age": [18, 25, 30, 45],
        "salary": [30000, 60000, 90000, 120000]
    }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is column in expected_dtypes?
5. Check value range
6. Validate each column's dtypeand range
7. If validation fails, raiseValueError
8. Print error message
9. End

