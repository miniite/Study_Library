---
type: "practice"
difficulty: "Deep"
---

# How would you version your Flask ML API so old clients keep working when the model input schema changes?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Loading Models Efficiently](<../../../../TOOLS/Python/Flask/Loading_Models_Efficiently.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To version a Flask ML API such that older clients continue to work even as the model's input schema evolves, we need to ensure backward compatibility while allowing for new features or changes in future updates. This involves carefully managing the API endpoints and their versions, possibly using query parameters to indicate which version of the API is being used by the client. Let’s break down the solution step-by-step.

#### Triage

The core question here is about maintaining compatibility between different clients (old vs new) while updating the model's input schema in a Flask application. The main concern is how to handle changes in the data structure without breaking existing clients that expect a specific format.

#### Assumptions
and Solution Overview

Assuming we have an existing Flask API serving machine learning models, we need to introduce versioning in such a way that:

Old clients continue to work with their expected input schema.
New clients can use updated schemas if necessary.
Future updates do not break the legacy client compatibility.
First Principles Approach
Versioning Strategy: Introduce API versioning using URL structure or query parameters. For example, api/v1/predict and api/v2/predict.
Input Schema Handling: Use a flexible input handling mechanism that can parse different versions of the input schema.
Legacy Support via Endpoint Mapping: Map old clients to specific endpoints that understand their version of the input.
Step-by-Step Solution
1. Define Versioned Endpoints

We define separate endpoints for each version:

PYTHON
# app.py
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/v1/predict', methods=['POST'])
def predict_v1():
    data = request.get_json()
    # Handle v1 input schema here
    return handle_prediction(data)

@app.route('/api/v2/predict', methods=['POST'])
def predict_v2():
    data = request.get_json()
    # Handle v2 input schema here
    return handle_prediction(data)
2. Flexible Input Handling

Create a function to handle the prediction and ensure it can parse both versions:

PYTHON
def handle_prediction(data):
    try:
        # Attempt to handle v1 input schema
        if "old_key" in data:  # Assuming v1 has this key
            result = process_v1_data(data)
        else:
            # Fallback for v2 or other future versions
            result = process_v2_data(data) if "new_key" in data else {"error": "Invalid input"}
    except Exception as e:
        return jsonify({"error": str(e)}), 400

    return jsonify(result)
3. Legacy Support via Endpoint Mapping

Ensure that old clients can still send requests to the v1 endpoint even if they are updated with a new client library or version.

PYTHON
@app.route('/predict', methods=['POST'])
def legacy_predict():
    # Redirects old clients to the appropriate versioned endpoint
    return predict_v1(), 301

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we have introduced a versioning strategy using separate endpoints for different API versions. This allows us to maintain backward compatibility while enabling future updates without breaking legacy clients. Some common mistakes include:

Not handling edge cases where the input schema is ambiguous or malformed.
Failing to redirect old clients properly to avoid breaking their functionality.

For follow-ups, we might explore more advanced strategies like using HTTP headers for versioning or leveraging modern frameworks that support automatic API versioning.

**Comparison:**

```
Input Schema	Version 1	Version 2
Key	old_key	new_key
```

**Code:**

```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/v1/predict', methods=['POST'])
def predict_v1():
    data = request.get_json()
    return handle_prediction(data)

@app.route('/api/v2/predict', methods=['POST'])
def predict_v2():
    data = request.get_json()
    return handle_prediction(data)

def handle_prediction(data):
    try:
        if "old_key" in data:  # Assuming v1 has this key
            result = process_v1_data(data)
        else:
            result = process_v2_data(data) if "new_key" in data else {"error": "Invalid input"}
    except Exception as e:
        return jsonify({"error": str(e)}), 400

    return jsonify(result)

@app.route('/predict', methods=['POST'])
def legacy_predict():
    # Redirects old clients to the appropriate versioned endpoint
    return predict_v1(), 301
```

**Example input / output:**

```

Sample input (legacy client):

JSON
Copy
{"old_key": 42}

Response:

JSON
Copy
{"prediction": 103.5}

Sample input (new client):

JSON
Copy
{"new_key": 7}

Response:

JSON
Copy
{"prediction": 84.6}
```

**Data:**

```json
{
  "old_key": {"value": 42},
  "new_key": {"value": 7}
}
```

**Flow:**

1. Yes
2. No
3. Client sends request
4. Is versioned endpoint?
5. Redirect to appropriate version
6. Send request to v1 endpoint
7. Parse input schema andprocess prediction

