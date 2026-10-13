---
type: "practice"
difficulty: "Deep"
---

# How do you validate that a FastAPI-served model gives the same output as the training environment (parity testing), and what tolerance do you allow for float differences?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure parity between a FastAPI-served machine learning model and its training environment, we need to validate that they produce similar outputs given identical inputs. This involves setting up tests that compare predictions from both environments and establishing an acceptable tolerance level for any floating-point differences.

First, let's break down the key aspects of this problem:

Environment Consistency: Ensuring that both the training and production environments are as close as possible.
Input Reproducibility: Generating identical inputs in both environments.
Output Comparison: Comparing model predictions from both environments.
Tolerance Level: Setting an acceptable range for differences between outputs.

Now, let's walk through a step-by-step solution to achieve this parity testing:

Step 1: Environment Setup

Both the training and production environments should be as similar as possible. This includes:

Python version compatibility.
Libraries and versions.
Hardware (if applicable).
Data preprocessing steps.
Step 2: Input Generation

Generate a set of test inputs that cover various scenarios, including edge cases. These inputs should be used in both environments to ensure they produce the same results.

Step 3: Model Prediction in Training Environment

Train your model and use it to predict outputs on the generated test inputs.

Step 4: Setting Up FastAPI

Deploy the trained model as a FastAPI service, serving predictions over an API.

PYTHON
from fastapi import FastAPI
import uvicorn

app = FastAPI()

@app.get("/predict")
def predict(input_data):
    # Load your model and make prediction here
    return {"prediction": 0.5}  # Example output
Step 5: Parity Testing

Compare the predictions from both environments using a test suite.

PYTHON
import requests

# Test inputs
test_inputs = [{"feature1": 1.0, "feature2": 2.0}, {"feature1": -1.0, "feature2": 3.0}]

def check_parity():
    # Predictions from the training environment
    training_outputs = [your_model.predict(input) for input in test_inputs]

    # Predictions from FastAPI service
    fastapi_outputs = []
    for input_data in test_inputs:
        response = requests.get("http://localhost:8000/predict", json=input_data)
        fastapi_outputs.append(response.json()["prediction"])

    # Calculate the tolerance level (e.g., 1e-4)
    tolerance_level = 1e-4

    for training_output, fastapi_output in zip(training_outputs, fastapi_outputs):
        if abs(training_output - fastapi_output) > tolerance_level:
            print(f"Parity failed: {training_output} != {fastapi_output}")
Step 6: Tolerance Level

The tolerance level is crucial because floating-point arithmetic can introduce small differences. Setting an appropriate tolerance ensures that minor discrepancies do not falsely indicate a failure.

For example, if the outputs are 0.498 and 0.502, they should be considered equal within a tolerance of 1e-3. However, setting this too high might miss significant issues, while setting it too low could lead to false positives.

#### Closing Summary

To ensure parity between your FastAPI-served model and the training environment, you need to:

Ensure both environments are consistent.
Generate identical test inputs.
Compare predictions from both environments.
Set an appropriate tolerance level for floating-point differences.

#### Common mistakes
include neglecting subtle differences in the setup (e.g., different Python versions), not covering edge cases adequately, and setting an overly strict or loose tolerance level.

