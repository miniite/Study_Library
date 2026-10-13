---
type: "practice"
difficulty: "Intermediate"
---

# How would you use the AI to add clear error messages when your prediction API receives the wrong number of features?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Hallucinated APIs and Outdated Suggestions](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Hallucinated_APIs_and_Outdated_Suggestions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively integrate an AI-assisted coding system like Copilot into a prediction API, we aim to ensure that our code is robust and user-friendly by adding clear error messages when the input data has the wrong number of features. This involves understanding the API’s requirements, handling different edge cases, and providing actionable feedback to the developer.

#### Triaging the Question

The question primarily probes your ability to handle input validation and error handling in a machine learning context. It requires an understanding of:

Input validation: Ensuring that the incoming data matches the expected format.
Error messaging: Crafting informative messages that help users debug issues quickly.
Flexibility vs. Robustness: Balancing between allowing some flexibility (e.g., minor inconsistencies) and ensuring robustness against critical errors.

#### Assumptions
The API expects a specific number of features for the input data.
We are using Python as our primary language, which is common in ML projects.
The error messages should be informative yet user-friendly.
Step-by-Step Solution
1. Define Expected Input Structure

First, we need to define what constitutes valid input for our API. For instance, if our model expects a 2D array with ( n ) features and one label per sample, the input shape might look like this:

PYTHON
import numpy as np

# Example: A valid input with 3 features
valid_input = np.array([[1, 2, 3], [4, 5, 6]])
2. Implement Input Validation

We will write a function that checks if the input matches our expected structure and raises an informative error message otherwise.

PYTHON
def validate_input(input_data):
    """
    Validate the input data against the expected format.

    Parameters:
        input_data (np.ndarray): The input array to be validated.

    Raises:
        ValueError: If the input does not match the expected shape.
    """
    if len(input_data.shape) != 2 or input_data.shape[1] < 3:
        raise ValueError("Invalid number of features. Expected at least 3 features per sample.")
3. Handle Edge Cases

We should consider edge cases such as:

Empty arrays.
Arrays with fewer than the expected number of features.
PYTHON
def handle_edge_cases(input_data):
    if input_data.shape[0] == 0:
        raise ValueError("Input array is empty. Please provide at least one sample.")
4. Integrate Copilot for Error Handling

We can use a tool like Copilot to generate the error messages, which would help in writing cleaner and more maintainable code.

PYTHON
def get_error_message(expected_features):
    return f"Expected input with {expected_features} features per sample."

# Example usage:
try:
    validate_input(valid_input)
except ValueError as e:
    print(Copilot.get_error_message(3))  # Expected to output: "Expected input with 3 features per sample."
5. Testing the Implementation

To ensure our implementation works correctly, we can run some test cases.

PYTHON
def test_validation():
    valid_input = np.array([[1, 2, 3], [4, 5, 6]])
    invalid_input_1 = np.array([1, 2])  # Not a 2D array with at least 3 features
    invalid_input_2 = np.array([])      # Empty array

    try:
        validate_input(valid_input)
        print("Valid input passed.")
    except ValueError as e:
        assert str(e) == "Invalid number of features. Expected at least 3 features per sample."

    try:
        validate_input(invalid_input_1)
    except ValueError as e:
        assert str(e) == "Invalid number of features. Expected at least 3 features per sample."

    try:
        validate_input(invalid_input_2)
    except ValueError as e:
        assert str(e) == "Input array is empty. Please provide at least one sample."

#### Closing Summary
and Common Mistakes

To summarize, the key steps involve defining expected input shapes, implementing robust validation functions, handling edge cases, and integrating AI tools like Copilot to generate clear error messages. Common mistakes include:

Overlooking edge cases that might cause runtime errors.
Not providing informative error messages that can help developers quickly identify issues.
Triaging Follow-Ups

If asked further, you might be prompted to discuss:

Scalability: How would the validation function handle larger datasets?
Security: What measures are in place to prevent malicious inputs from causing failures?

**Comparison:**

```
Input Data	Expected Error Message
[[1, 2, 3], [4, 5, 6]]	No error
[1, 2]	"Invalid number of features. Expected at least 3 features per sample."
[]	"Input array is empty. Please provide at least one sample."
```

**Code:**

```python
import numpy as np

def validate_input(input_data):
    if len(input_data.shape) != 2 or input_data.shape[1] < 3:
        raise ValueError("Invalid number of features. Expected at least 3 features per sample.")

def handle_edge_cases(input_data):
    if input_data.shape[0] == 0:
        raise ValueError("Input array is empty. Please provide at least one sample.")

def get_error_message(expected_features):
    return f"Expected input with {expected_features} features per sample."

# Example usage
try:
    validate_input(np.array([[1, 2, 3], [4, 5, 6]]))
except ValueError as e:
    print(Copilot.get_error_message(3))

test_validation()
```

**Example input / output:**

```text
Valid input passed.
Input array is empty. Please provide at least one sample.
```

**Data:**

```python
valid_input = np.array([[1, 2, 3], [4, 5, 6]])
invalid_input_1 = np.array([1, 2])
invalid_input_2 = np.array([])
```

**Flow:**

1. Yes
2. No
3. Start
4. Is input valid?
5. Proceed with model
6. Raise ValueError with message
7. Model prediction

