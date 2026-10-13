---
type: "practice"
difficulty: "Intermediate"
---

# How would you cache predictions for repeated identical inputs in a Flask ML API to save computation?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Loading Models Efficiently](<../../../../TOOLS/Python/Flask/Loading_Models_Efficiently.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address caching predictions for repeated identical inputs in a Flask-based Machine Learning (ML) API, we need to ensure that when the same input data is sent multiple times, the model's prediction does not have to be recomputed from scratch every time. This approach significantly reduces computation and improves response time.

#### Triaging the Question

The core of this question revolves around implementing a caching mechanism within a Flask application to store the results of predictions for previously processed inputs. The interviewer is probing your understanding of:

Caching Concepts: How do you design a cache to efficiently store and retrieve data?
Flask Integration: How can you integrate caching into an existing Flask API without affecting its functionality?

#### Performance Considerations
What are the trade-offs between memory usage and retrieval speed?

#### Assumptions
The input data is in a form that can be stored (e.g., JSON, numpy array).
You have a basic understanding of how to run predictions using your ML model.
You need to store results only for specific input types or conditions.
Step-by-Step Solution
1. Choose a Caching Strategy

For an intermediate level, we can use the functools.lru_cache decorator which is part of Python's standard library and provides a simple way to implement function caching. This approach is straightforward and effective for our needs.

PYTHON
from flask import Flask, request, jsonify
import functools
import numpy as np

app = Flask(__name__)

# Example ML model prediction function (simplified)
def predict(input_data):
    # Simplified: assume this predicts a single output based on input_data
    return np.dot(np.array(input_data), [1, 2, 3])

# Caching predictions using functools.lru_cache
@functools.lru_cache(maxsize=100)  # Cache up to 100 unique inputs
def cached_predict(input_data):
    return predict(input_data)
2. Integrate with Flask API

We will integrate the caching function into our Flask API route.

PYTHON
@app.route('/predict', methods=['POST'])
def api_predict():
    input_data = request.get_json()

    # Ensure the data is in a suitable format for prediction
    if not isinstance(input_data, list):
        return jsonify({"error": "Input must be a list"}), 400

    result = cached_predict(tuple(input_data))
    return jsonify({"prediction": result.tolist()})
3. Handle Edge Cases and Failures
Ensure input data is always in the expected format (a list).
Implement error handling for cache misses or other potential issues.

#### Closing Summary

By using functools.lru_cache, we can efficiently store and retrieve predictions from a Flask API, reducing unnecessary computation. This approach balances simplicity with performance gains, making it suitable for many intermediate-level projects.

#### Common Mistakes/Follow-Ups
Cache Misses: Ensure you handle cases where the cache is not available.
Memory Management: Be mindful of memory usage, especially when using large caches.
Input Validation: Always validate input data before passing it to your ML model to avoid unexpected behavior.

