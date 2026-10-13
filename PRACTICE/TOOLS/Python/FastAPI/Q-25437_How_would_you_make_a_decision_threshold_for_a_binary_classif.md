---
type: "practice"
difficulty: "Basic"
---

# How would you make a decision threshold for a binary classifier configurable through a FastAPI query parameter?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Time-series Prediction APIs](<../../../../TOOLS/Python/FastAPI/Time-series_Prediction_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up a configurable decision threshold for a binary classifier in FastAPI, we need to expose an API endpoint that allows users to input their desired threshold as a query parameter. This involves understanding how classifiers work and FastAPI's capabilities for handling HTTP requests and responses.

Firstly, let’s clarify what we are building:

Objective: Create an API where users can specify a decision threshold.
Technology Stack: FastAPI for the backend.
Flow: Users will make a GET request with a query parameter, and our API will return predictions based on this threshold.

#### Triaging the Question

The core of the question is about dynamically adjusting the decision boundary (threshold) used by a binary classifier at runtime. We need to ensure that:

The FastAPI server can accept and parse user inputs.
Our model can use these inputs effectively without losing state or performance.

#### Assumptions
include:

You have a working machine learning pipeline with a trained model.
Your data is preprocessed in a way that it can be fed into the model.

#### First Principles Solution

Let's break down the process:

Model Setup: Ensure your ML model (e.g., scikit-learn, TensorFlow) has a predict_proba method if using probability scores or another method to get decision values.
FastAPI Endpoint Creation: Design an endpoint that accepts a query parameter for the threshold.
Prediction Logic: Use the provided threshold in the prediction logic.

#### Implementation

Install FastAPI and Dependencies:

BASH
pip install fastapi uvicorn

Create Your FastAPI Application:

PYTHON
from fastapi import FastAPI, Query
import numpy as np  # Example: Using NumPy for demonstration purposes
from sklearn.linear_model import LogisticRegression  # Example classifier

app = FastAPI()

# Sample model and data
model = LogisticRegression()
X_test = np.array([[1.0, 2.0], [3.0, 4.0]])  # Placeholder test data
y_pred_proba = model.predict_proba(X_test)

@app.get("/predict/")
async def predict(threshold: float = Query(0.5)):
    """
    Predicts binary class labels based on a provided decision threshold.

    Parameters:
    - threshold (float): Decision threshold for the classifier.

    Returns:
    A list of predicted classes (0 or 1) based on the given threshold.
    """
    y_pred = np.where(y_pred_proba[:, 1] > threshold, 1, 0)
    return {"threshold": threshold, "predictions": y_pred.tolist()}

#### Closing Summary
and Common Mistakes/Follow-Ups

This FastAPI setup provides a simple yet effective way to dynamically adjust the decision threshold for binary classification. Key takeaways include:

Ensuring your model supports probability predictions or an equivalent.
Handling query parameters correctly in FastAPI.
Using np.where effectively to convert probabilities into class labels.

#### Common mistakes
might include:

Not properly handling edge cases (e.g., invalid thresholds).
Failing to preprocess data appropriately before passing it to the model.
Overlooking type validation or error handling for query parameters.

For a more robust solution, consider adding validation checks and logging mechanisms to handle errors gracefully and ensure performance optimization in production.

