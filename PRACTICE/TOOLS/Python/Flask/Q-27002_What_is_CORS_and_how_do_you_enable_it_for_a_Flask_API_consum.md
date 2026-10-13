---
type: "practice"
difficulty: "Intermediate"
---

# What is CORS and how do you enable it for a Flask API consumed by a frontend?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Cross-Origin Resource Sharing (CORS) is a security feature that prevents web applications from making requests to a different domain than the one serving the application. In the context of Flask, enabling CORS means configuring your API to allow specific origins to make requests to it.

The question revolves around understanding CORS and implementing its configuration in a Flask backend for a frontend application. We need to ensure secure communication between our frontend and backend while allowing necessary cross-origin access.

#### Triaging the Question
What is CORS?
CORS is a security mechanism that ensures web applications running on one domain can only make requests (HTTP/HTTPS) to another domain if explicitly allowed by the server.
Why do we need CORS in Flask?
Frontend applications often run on different domains or subdomains than their backend services. To allow these applications to communicate with our API, we must configure CORS settings.
Key Assumptions:
The frontend is hosted on a different domain (e.g., http://localhost:3000).
We want to control which origins can make requests to our Flask backend.

#### First-Principles Solution

To enable CORS in a Flask application, we will use the flask-cors extension. This library provides an easy way to configure and manage CORS policies for your API endpoints.

Step 1: Install Flask-CORS

First, install the flask-cors package via pip:

BASH
pip install flask-cors
Step 2: Configure CORS in Flask

Next, we need to import CORS from flask_cors and configure it for our Flask application. Here's a step-by-step guide:

Import the Necessary Libraries:

PYTHON
from flask import Flask
from flask_cors import CORS

Initialize the Flask Application:

PYTHON
app = Flask(__name__)

Enable CORS with Specific Origins:

Let's say our frontend is hosted on http://localhost:3000. We can enable CORS for this specific origin:
PYTHON
cors = CORS(app, resources={r"/*": {"origins": "http://localhost:3000"}})

Define and Run the Flask Application:

PYTHON
@app.route('/')
def home():
    return 'Welcome to the API!'

if __name__ == '__main__':
    app.run(debug=True)
Step 3: Testing CORS Configuration

To test our CORS configuration, we can use a simple cURL command from the frontend:

SH
curl -X GET "http://localhost:5000/"

Ensure that this request is successful and returns the expected response.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: We've enabled CORS in our Flask backend by using the flask-cors extension. This allows us to specify which origins can make requests to our API, ensuring secure communication between the frontend and backend.

#### Common Mistakes
Not properly configuring CORS settings can lead to security vulnerabilities.
Failing to test CORS configurations thoroughly might result in unexpected errors when the application goes live.

#### Follow-Ups
The interviewer may ask about handling wildcard origins (*), which should be used cautiously as it allows all origins. They might also inquire about how to handle different HTTP methods (GET, POST, etc.) or headers.

**Code:**

```python
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)
cors = CORS(app, resources={r"/*": {"origins": "http://localhost:3000"}})

@app.route('/')
def home():
    return 'Welcome to the API!'

if __name__ == '__main__':
    app.run(debug=True)
```

**Flow:**

1. Yes
2. No
3. Home Route
4. Is Origin Allowed?
5. Return Response
6. 403 Forbidden or 401Unauthorized

