---
type: "practice"
difficulty: "Intermediate"
---

# How would you attach a unique request id to each LLM call and thread it through logs in a Flask app for tracing?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Streaming Responses](<../../../../TOOLS/Python/Flask/Streaming_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that every API call to an LLM (Language Model) within a Flask application is uniquely traceable, we'll attach a unique request ID to each incoming request and thread it through all logs. This will help in tracing the flow of requests and identifying issues more effectively.

#### Triaging the Question

The question essentially requires us to implement a mechanism where every API call to our LLM gets a unique identifier (request ID) that is preserved throughout the processing pipeline and logged for traceability. The Flask app should handle this request ID in such a way that it can be used at any point during the request handling process.

#### Assumptions
We assume the application uses Flask as its web framework.
Each incoming API call to the LLM will include this unique request ID via HTTP headers or query parameters.
The logging mechanism should capture this ID for each step of processing and output it in a structured format.

#### First-Principles Solution
Step 1: Middleware for Request ID Handling

We can create a middleware function that attaches the request ID to the Flask application context before any route handler is called. This ensures the ID is available throughout the request lifecycle.

PYTHON
from flask import Flask, g, request

app = Flask(__name__)

def attach_request_id(middleware):
    @middleware.app.before_request
    def before_request():
        # Generate a unique request ID if it's not already set.
        if 'request_id' not in request.headers:
            request_id = generate_unique_request_id()
            request.headers['X-Request-ID'] = request_id

def generate_unique_request_id():
    # For simplicity, we can use an UUID generator
    import uuid
    return str(uuid.uuid4())

# Middleware registration
attach_request_id(middleware=app)

@app.route('/llm', methods=['POST'])
def llm_endpoint():
    # Get the unique request ID from the headers
    request_id = request.headers.get('X-Request-ID')

    # Log or store this for traceability
    print(f"Handling LLM call with Request ID: {request_id}")

    # Process the request...
    response = process_llm_request(request.data)

    return {'status': 'success', 'response': response}
Step 2: Logging Mechanism

Ensure that every log statement within your application includes the unique request ID. Flask's logging can be configured to include this information.

PYTHON
import logging

log = logging.getLogger('werkzeug')
log.setLevel(logging.DEBUG)

# Log a sample message with request_id
logging.info(f"Request processed with ID: {request_id}")

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key steps are to:

Generate a unique request ID for each incoming API call.
Attach this ID to the Flask application context using middleware.
Ensure that logs capture and include this ID.

#### Common mistakes
include not handling edge cases where the request ID might be missing or invalid. Always validate and handle these scenarios gracefully.

#### Follow-up
questions could involve optimizing performance, considering security implications (e.g., protecting against injection attacks), and integrating with distributed tracing systems like Jaeger for more comprehensive traceability.

**Code:**

```python
from flask import Flask, g, request

app = Flask(__name__)

def attach_request_id(middleware):
    @middleware.app.before_request
    def before_request():
        if 'request_id' not in request.headers:
            request_id = generate_unique_request_id()
            request.headers['X-Request-ID'] = request_id

def generate_unique_request_id():
    import uuid
    return str(uuid.uuid4())

# Middleware registration
attach_request_id(middleware=app)

@app.route('/llm', methods=['POST'])
def llm_endpoint():
    request_id = request.headers.get('X-Request-ID')

    print(f"Handling LLM call with Request ID: {request_id}")

    response = process_llm_request(request.data)

    return {'status': 'success', 'response': response}

@app.route('/health', methods=['GET'])
def health_check():
    return {'status': 'up'}
```

**Example input / output:**

```text
{
  "status": "success",
  "response": {
    "message": "LLM call processed with Request ID 123e4567-e89b-12d3-a456-426614174000"
  }
}
```

**Flow:**

1. No
2. Yes
3. Receive Request
4. Is 'X-Request-ID' present?
5. Generate Unique ID
6. Set 'X-Request-ID' header
7. Use existing 'X-Request-ID'
8. F
9. Process LLM Request
10. Return Response

