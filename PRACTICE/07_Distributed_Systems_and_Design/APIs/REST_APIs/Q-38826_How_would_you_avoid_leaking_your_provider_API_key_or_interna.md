---
type: "practice"
difficulty: "Intermediate"
---

# How would you avoid leaking your provider API key or internal error details in the error responses your AI API sends to clients?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Keeping Secrets out of URLs](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Keeping_Secrets_out_of_URLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that our AI API does not leak sensitive information like the provider's API key or detailed error messages, we need to design robust and secure error handling mechanisms. The core idea is to provide meaningful but generic error responses to clients while safeguarding sensitive details.

#### Triage

The question asks how to prevent exposing internal errors and credentials in REST API error responses. This involves understanding what information should be exposed vs. hidden, considering security best practices, and ensuring that the client receives only necessary information.

#### Assumptions
We are using a standard RESTful API.
The API is built using a framework like Flask or Express.js.
The provider's API key is stored securely (e.g., in environment variables).
Error responses follow a consistent structure.

#### Solution
Step 1: Define Error Response Structure

First, define a generic error response schema that includes an HTTP status code and a message. Avoid including stack traces or detailed error logs.

JSON
{
    "error": {
        "code": 500,
        "message": "Internal Server Error"
    }
}
Step 2: Implement Error Handling in the Backend

In your backend code, catch exceptions and return generic error responses. Ensure sensitive information is not included.

#### Example
with Flask:

PYTHON
from flask import jsonify

@app.errorhandler(Exception)
def handle_exception(e):
    response = {
        "error": {
            "code": 500,
            "message": "Internal Server Error"
        }
    }
    return jsonify(response), 500
Step 3: Use Environment Variables for Credentials

Store your provider's API key in environment variables to avoid hardcoding it in the source code.

BASH
export PROVIDER_API_KEY='your_api_key_here'

#### Example
with Flask:

PYTHON
import os

provider_api_key = os.getenv('PROVIDER_API_KEY')

@app.route('/predict', methods=['POST'])
def predict():
    try:
        # Use provider_api_key safely
        pass
    except Exception as e:
        response = {
            "error": {
                "code": 500,
                "message": "Failed to process request. Please try again."
            }
        }
        return jsonify(response), 500
Step 4: Client-Side Error Handling

Ensure the client handles error responses gracefully and does not rely on detailed server-side information.

#### Example
with JavaScript:

JAVASCRIPT
fetch('/predict', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' }
})
.then(response => response.json())
.catch(error => console.error('Error:', error))

#### Closing Summary

By implementing generic error responses, using environment variables to store sensitive information, and ensuring proper backend and frontend handling, we can protect our API from leaking internal details. This approach balances usability with security.

#### Common Mistakes/Follow-Ups
Mistake: Returning detailed stack traces or error logs in error responses.

#### Follow-Up
Discuss potential attacks like error-based SQL injection where specific error messages might help an attacker exploit the system.

**Code:**

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.errorhandler(Exception)
def handle_exception(e):
    response = {
        "error": {
            "code": 500,
            "message": "Internal Server Error"
        }
    }
    return jsonify(response), 500

@app.route('/predict', methods=['POST'])
def predict():
    try:
        # Use provider_api_key safely
        pass
    except Exception as e:
        response = {
            "error": {
                "code": 500,
                "message": "Failed to process request. Please try again."
            }
        }
        return jsonify(response), 500

if __name__ == '__main__':
    app.run()
```

**Example input / output:**

```text
# Example Error Response from Client-Side Code
{
    "error": {
        "code": 500,
        "message": "Internal Server Error"
    }
}
```

**Flow:**

1. Yes
2. No
3. Client Request
4. API Endpoint
5. Exception?
6. Generic Error Response
7. Successful Response
8. Log Internal Error forDebugging

