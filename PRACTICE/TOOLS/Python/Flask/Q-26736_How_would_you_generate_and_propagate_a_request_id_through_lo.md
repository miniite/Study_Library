---
type: "practice"
difficulty: "Deep"
---

# How would you generate and propagate a request id through logs using a before_request hook in Flask?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate and propagate a request ID through logs using a before_request hook in Flask, we'll follow these steps:

#### Overview
We need to ensure that every incoming HTTP request receives a unique identifier (request ID) which is then included in the logs for traceability.

#### Triaging the Question
The key here is understanding how to inject the request ID into the logging mechanism and maintain it throughout the request lifecycle. We will use Flask's before_request hook to generate this ID, store it in a thread-local object (to ensure it persists within one request), and pass it through our application logic.
Step-by-Step Solution:
Use Flask’s before_request decorator to create an endpoint that generates the request ID.
Store this ID in the flask.g global object, which is thread-safe for storing data during a single request.
Pass this ID through each subsequent handler and log it where necessary.

Here's how we can implement this:

Code Implementation:
PYTHON
from flask import Flask, request, g

app = Flask(__name__)

@app.before_request
def generate_request_id():
    if 'X-Request-ID' in request.headers:
        g.request_id = request.headers['X-Request-ID']
    else:
        import uuid
        g.request_id = str(uuid.uuid4())
        # Optionally set this ID as a header to propagate it further, e.g., for logs
        response = app.make_response(request.environ)
        response.headers['X-Request-ID'] = g.request_id

@app.route('/')
def index():
    log(f"Processing request with id: {g.request_id}")
    return "Hello World!"

if __name__ == '__main__':
    app.run()

#### Closing Summary
By using before_request, we ensure that every incoming request gets a unique identifier stored in the thread-local object, which is then passed to subsequent handlers and eventually logged.

#### Common Mistakes/Follow-Ups

Not propagating the ID properly through all components of your application.
Failing to handle cases where X-Request-ID is not present in the headers (e.g., during initial requests).
Overlooking potential performance implications due to creating a new UUID for every request.

