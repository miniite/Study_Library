---
type: "practice"
difficulty: "Basic"
---

# How would you calculate the error rate of a service in Python given a total request count and an error count, and express it as a percentage?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Percentage Change Calculations](<../../../../TOOLS/Python/Python_Language/Percentage_Change_Calculations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the error rate of a service in Python, we need to determine the proportion of requests that resulted in errors out of the total number of requests. This can be expressed as a percentage by multiplying the result by 100. Here’s how you would approach this:

First, let's break down what the question is asking:

We are given two pieces of data: the total request count and the error count.
The goal is to compute the error rate and express it as a percentage.

Now, we will reason through the steps from first principles. We start by defining our variables:

Let total_requests be the total number of requests made to the service.
Let error_count be the number of requests that resulted in errors.

The error rate can be calculated using the formula: [ \text{Error Rate} = \left( \frac{\text{error_count}}{\text{total_requests}} \right) \times 100 ]

This gives us a percentage value representing the proportion of requests that were erroneous. Let's implement this in Python:

PYTHON
def calculate_error_rate(total_requests, error_count):
    """
    Calculate the error rate as a percentage.

    Args:
        total_requests (int): Total number of requests.
        error_count (int): Number of errors.

    Returns:
        float: Error rate as a percentage.
    """
    if total_requests == 0:
        return 0.0  # Avoid division by zero
    error_rate = (error_count / total_requests) * 100
    return error_rate

# Example usage
total_requests = 1000
error_count = 50
error_rate_percentage = calculate_error_rate(total_requests, error_count)
print(f"Error Rate: {error_rate_percentage:.2f}%")

This function takes the total number of requests and the number of errors as input and returns the error rate as a percentage. We handle edge cases where total_requests might be zero to avoid division by zero.

**Code:**

```python
def calculate_error_rate(total_requests, error_count):
    """
    Calculate the error rate as a percentage.

    Args:
        total_requests (int): Total number of requests.
        error_count (int): Number of errors.

    Returns:
        float: Error rate as a percentage.
    """
    if total_requests == 0:
        return 0.0  # Avoid division by zero
    error_rate = (error_count / total_requests) * 100
    return error_rate

# Example usage
total_requests = 1000
error_count = 50
error_rate_percentage = calculate_error_rate(total_requests, error_count)
print(f"Error Rate: {error_rate_percentage:.2f}%")
```

**Example input / output:**

```text
Error Rate: 5.00%
```

