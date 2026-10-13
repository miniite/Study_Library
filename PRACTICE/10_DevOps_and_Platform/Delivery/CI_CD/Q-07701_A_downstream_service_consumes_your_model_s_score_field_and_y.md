---
type: "practice"
difficulty: "Deep"
---

# A downstream service consumes your model's score field and you changed its range from 0-1 to logits; how would CI have caught this contract break?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a contract break between your ML model's score field and a downstream service, we need to ensure that our Continuous Integration/Continuous Deployment (CI/CD) pipeline includes comprehensive checks to validate the data contract. The primary concern here is ensuring that any changes in how the model outputs its scores are accurately reflected in the downstream service’s expectations.

The question at hand is: How would CI have caught this change from a 0-1 score range to logits? This involves understanding the concept of contracts, specifically input-output contracts, and how we can design our CI/CD pipeline to enforce these contracts. We will assume that both the model output and downstream service expect specific ranges or types of data.

Let's break down the solution step-by-step:

#### Triaging the Question

The core issue is a change in the expected format of the model’s score field, from being directly interpretable as probabilities (0-1) to logits. This change could introduce discrepancies between the model output and the downstream service expectations, leading to potential failures or bugs.

#### Assumptions

The model's output now produces logits.
Downstream services expect probabilities in the range [0, 1].
CI/CD is responsible for validating these contracts.

#### First-Principles Solution

To catch such a contract break during CI:

Define the Data Contract: Clearly define what the model should produce and how downstream systems will use this data.
Add Unit Tests in CI Pipeline: Create tests that validate the model's output against expected ranges or formats.
Incorporate Model Output Validation: Use tools like pytest (for Python) to write unit tests that check if the model’s logits are correctly transformed into probabilities.
Use Mocks and Stubs: Employ mocks for downstream services during local testing to ensure that any changes in output format do not break existing integration points.
Detailed Steps

Define Expected Output:

The model should output a list of logits.
Downstream service expects a list of probabilities in the range [0, 1].

Create Unit Tests for Model:

PYTHON
import numpy as np
from sklearn.linear_model import LogisticRegression

# Sample model
model = LogisticRegression()

def test_output_range():
    # Generate some input data
    X = np.random.rand(10, 5)
    y = [1] * 5 + [0] * 5

    # Fit the model
    model.fit(X, y)

    # Predict logits
    logits = model.predict_log_proba(X)

    # Check if logits are within a reasonable range (e.g., -2 to 2)
    assert np.all(logits >= -2) and np.all(logits <= 2), "Logits out of expected range"

def test_output_transform():
    # Generate some input data
    X = np.random.rand(10, 5)
    y = [1] * 5 + [0] * 5

    # Fit the model
    model.fit(X, y)

    # Predict logits and transform to probabilities
    logits = model.predict_log_proba(X)
    probs = softmax(logits)

    # Check if all probabilities are within the range [0, 1]
    assert np.all(probs >= 0) and np.all(probs <= 1), "Probabilities out of expected range"

Integrate with CI Pipeline:

Add these tests to your CI pipeline using tools like Jenkins, GitLab CI, or GitHub Actions.
Ensure that every commit triggers a full run of the tests.

Mock Downstream Services:

PYTHON
from unittest.mock import Mock

def mock_downstream_service():
    service = Mock()
    # Simulate expected behavior
    return service

test_output_integration(mock_downstream_service())

#### Closing Summary
and Common Mistakes

In summary, to catch this contract break during CI, we need to ensure that our unit tests cover both the model's output range and transformation into probabilities. This involves defining clear data contracts, writing thorough tests, and integrating these checks into our CI pipeline.

#### Common mistakes
include:

Failing to define clear expectations for the model’s outputs.
Not incorporating robust validation in unit tests.
Ignoring edge cases during testing, such as extremely high or low logits.

By addressing these areas, we can ensure that changes like this are caught early and do not lead to downstream issues.

**Code:**

```python
import numpy as np
from sklearn.linear_model import LogisticRegression

# Sample model
model = LogisticRegression()

def test_output_range():
    # Generate some input data
    X = np.random.rand(10, 5)
    y = [1] * 5 + [0] * 5

    # Fit the model
    model.fit(X, y)

    # Predict logits
    logits = model.predict_log_proba(X)

    # Check if logits are within a reasonable range (e.g., -2 to 2)
    assert np.all(logits >= -2) and np.all(logits <= 2), "Logits out of expected range"

def test_output_transform():
    # Generate some input data
    X = np.random.rand(10, 5)
    y = [1] * 5 + [0] * 5

    # Fit the model
    model.fit(X, y)

    # Predict logits and transform to probabilities
    logits = model.predict_log_proba(X)
    probs = softmax(logits)

    # Check if all probabilities are within the range [0, 1]
    assert np.all(probs >= 0) and np.all(probs <= 1), "Probabilities out of expected range"

def mock_downstream_service():
    service = Mock()
    return service

def test_output_integration(mock_service):
    # Perform integration tests using mocks
    pass

def softmax(logits):
    # Implement softmax function
    pass
```

