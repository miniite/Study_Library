---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate that an incoming prediction request has exactly the numeric features your model expects before calling model.predict?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Loading Models Efficiently](<../../../../TOOLS/Python/Flask/Loading_Models_Efficiently.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an incoming prediction request contains exactly the numeric features expected by a model, we can leverage Flask to preprocess and validate requests before passing them to our prediction function. This involves checking if all necessary features are present in the input data and handling any missing or unexpected features gracefully.

#### Triaging the Question

The core of this question is about validating incoming JSON payloads in a Flask application. We need to ensure that each request contains the exact set of features required by the model, and we should handle cases where some features are missing or additional features are present. The interviewer might be probing your understanding of Flask routing, JSON validation, and error handling.

#### First-Principles Solution
Understand the Model Requirements: Define which features your model expects.
Parse Incoming Requests: Use Flask to parse incoming JSON data.
Validate Features: Check if all required features are present in the request payload.
Handle Missing or Extra Features: Provide appropriate error messages and response handling.
Step-by-Step Solution

Define Expected Features:

Let's assume our model expects 5 numeric features: feature1, feature2, feature3, feature4, and feature5.

Create a Flask Route for Prediction Requests:

PYTHON
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/predict', methods=['POST'])
def predict():
    # Parse incoming JSON data
    input_data = request.json

    # Validate features
    required_features = {'feature1', 'feature2', 'feature3', 'feature4', 'feature5'}
    missing_features = required_features - set(input_data.keys())

    if missing_features:
        return jsonify({"error": f"Missing required features: {list(missing_features)}"}), 400

    # Check for unexpected features
    extra_features = set(input_data.keys()) - required_features
    if extra_features:
        return jsonify({"error": f"Unexpected features found: {list(extra_features)}"}), 400

    # Process and predict using the model
    prediction_result = model.predict([input_data.values()])
    return jsonify({"prediction": prediction_result.tolist()})

Error Handling:

If any required feature is missing, return a 400 Bad Request with an appropriate error message.
If there are any extra features present, also return a 400 Bad Request but indicate the presence of unexpected features.

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that only valid requests reach the model for prediction. By validating the input data, we prevent runtime errors and provide clear feedback to the client about what went wrong in their request. Some common mistakes include not handling missing or extra features gracefully or incorrectly interpreting the structure of the incoming JSON payload.

#### Follow-ups
might involve discussing how to handle large datasets efficiently, integrating this validation with more complex business logic, or scaling this solution for a high-traffic application.

**Comparison:**

```
Feature	Value
feature1	1.2
feature2	3.5
feature3	4.8
feature4	6.1
feature5	7.3
```

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/predict', methods=['POST'])
def predict():
    input_data = request.json

    required_features = {'feature1', 'feature2', 'feature3', 'feature4', 'feature5'}

    missing_features = required_features - set(input_data.keys())

    if missing_features:
        return jsonify({"error": f"Missing required features: {list(missing_features)}"}), 400

    extra_features = set(input_data.keys()) - required_features

    if extra_features:
        return jsonify({"error": f"Unexpected features found: {list(extra_features)}"}), 400

    prediction_result = model.predict([input_data.values()])

    return jsonify({"prediction": prediction_result.tolist()})
```

**Example input / output:**

```text
# Example Input Data
input_data = {
    "feature1": 1.2,
    "feature2": 3.5,
    "feature3": 4.8,
    "feature4": 6.1,
    "feature5": 7.3
}

# Expected Output
response = predict(input_data)
print(response)  # {"prediction": [0.9]}
```

**Data:**

```json
{
    "feature1": 1.2,
    "feature2": 3.5,
    "feature3": 4.8,
    "feature4": 6.1,
    "feature5": 7.3
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Receive Prediction Request
6. Parse JSON Data
7. Check Missing Features
8. Return Error (400)
9. Check Extra Features
10. Return Error (400)
11. Predict Using Model
12. Process and Return Prediction
13. Predict Using Model
14. End with 400 Bad Request
15. Return Prediction Result

