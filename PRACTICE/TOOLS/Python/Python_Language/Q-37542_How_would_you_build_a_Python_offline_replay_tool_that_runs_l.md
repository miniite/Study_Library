---
type: "practice"
difficulty: "Deep"
---

# How would you build a Python offline replay tool that runs logged production requests through a candidate prompt or model and diffs the outputs, so you catch regressions before deploy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Offline Evaluation Harnesses](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/Offline_Evaluation_Harnesses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build an offline replay tool in Python to catch regression issues before deployment, we need to log production requests, run them through a candidate model or prompt, and then compare the outputs against expected results. This involves several steps from logging and storage to processing and comparison.

#### Triaging the Question

The core of this question is about creating an automated testing framework that can replay historical data, execute it with new models, and ensure consistency in output. The key aspects include:

Logging Production Requests: How to capture requests accurately.
Model Execution: Running these captured requests through a new model or prompt.
Output Comparison: Comparing the model outputs against known correct results.
Scalability and Efficiency: Ensuring the solution is robust, efficient, and scalable.

#### Assumptions
We have access to historical production data logs.
The candidate models are versioned and can be reloaded for testing.
There's a mechanism to store expected output labels or reference results.
Step-by-Step Solution

Logging Production Requests:

Use Python’s built-in logging mechanisms (logging module) to capture requests.
Ensure the logs include all necessary metadata: request ID, timestamp, input data, and actual response (if known).

Model Execution:

Implement a function that loads the candidate model or prompt.
Write a handler that takes an input request from the log and processes it through the model.

Output Comparison:

Store expected outputs in a database or file for comparison purposes.
Develop a diffing mechanism to compare actual vs. expected outputs, flagging any discrepancies.

Scalability and Efficiency:

Use efficient data structures and algorithms.
Optimize model execution speed if necessary using techniques like caching, parallel processing, etc.
Design the tool to handle large volumes of logs efficiently.

#### Implementation

Here’s a high-level overview of the Python implementation:

PYTHON
import logging
from datetime import datetime
from sklearn.model_selection import train_test_split  # Example model for demonstration

# Logger setup
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Sample data loading (replace with actual production logs)
data = load_production_logs()  # Assume this function loads the logs

# Model definition and loading (this is a placeholder)
def load_model(version):
    if version == 'v1':
        return train_test_split
    elif version == 'v2':
        return some_other_model_function  # Placeholder for actual model
    else:
        raise ValueError("Unknown model version")

# Handler to process requests through the model
def process_request(request, model):
    input_data = request['input']
    output = model(input_data)
    logging.info(f"Processed {request['id']} with output: {output}")
    return output

# Diffing mechanism
def diff_outputs(expected_output, actual_output):
    if expected_output != actual_output:
        logging.warning(f"Mismatch detected in request {request['id']}: Expected {expected_output}, got {actual_output}")
    else:
        logging.info(f"Request {request['id']} passed: {expected_output} == {actual_output}")

# Main loop to replay and compare
for request in data:
    model = load_model('v2')  # Load the candidate model version
    actual_output = process_request(request, model)
    expected_output = get_expected_output(request)  # Assume this function retrieves expected output
    diff_outputs(expected_output, actual_output)

#### Closing Summary
and Common Mistakes/Follow-Ups

The above solution provides a basic structure for building an offline replay tool. Key points to consider include:

Efficient Logging: Ensure logs are clean and well-formatted.
Scalable Model Execution: Optimize execution time, especially if dealing with large datasets.
Comprehensive Error Handling: Add more robust error handling to manage unexpected scenarios.

#### Common mistakes
include not properly logging or parsing request data, failing to efficiently handle model execution, and not thoroughly testing the diffing mechanism. Follow-ups could involve discussing specific optimizations or trade-offs in different use cases.

**Comparison:**

```
Log ID	Model Version	Input Data	Expected Output	Actual Output	Status
1	v2	[1, 2]	[3]	[3]	Passed
2	v2	[4, 5]	[6]	[6]	Passed
```

**Code:**

```python
import logging
from datetime import datetime
from sklearn.model_selection import train_test_split  # Example model for demonstration

# Logger setup
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

# Sample data loading (replace with actual production logs)
def load_production_logs():
    return [
        {'id': '1', 'input': [1, 2], 'output': [3]},
        {'id': '2', 'input': [4, 5], 'output': [6]}
    ]

# Model definition and loading (this is a placeholder)
def load_model(version):
    if version == 'v1':
        return train_test_split
    elif version == 'v2':
        return some_other_model_function  # Placeholder for actual model
    else:
        raise ValueError("Unknown model version")

# Handler to process requests through the model
def process_request(request, model):
    input_data = request['input']
    output = model(input_data)
    logging.info(f"Processed {request['id']} with output: {output}")
    return output

# Diffing mechanism
def diff_outputs(expected_output, actual_output):
    if expected_output != actual_output:
        logging.warning(f"Mismatch detected in request {request['id']}: Expected {expected_output}, got {actual_output}")
    else:
        logging.info(f"Request {request['id']} passed: {expected_output} == {actual_output}")

# Main loop to replay and compare
for request in load_production_logs():
    model = load_model('v2')  # Load the candidate model version
    actual_output = process_request(request, model)
    expected_output = get_expected_output(request)  # Assume this function retrieves expected output
    diff_outputs(expected_output, actual_output)
```

**Example input / output:**

| Request ID | Input Data | Expected Output | Actual Output |
|------------|------------|-----------------|---------------|
| 1          | [1, 2]     | [3]             | [3]           |
| 2          | [4, 5]     | [6]             | [6]           |

**Data:**

```json
[
    {"id": "1", "input": [1, 2], "output": [3]},
    {"id": "2", "input": [4, 5], "output": [6]}
]
```

**Flow:**

1. Rendering diagram…

