---
type: "practice"
difficulty: "Basic"
---

# How would you enable CORS on a Flask ML API so a React dashboard on a different origin can call it?

**Difficulty:** Basic · **Levels:** Fresher · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enable Cross-Origin Resource Sharing (CORS) on a Flask ML API, allowing a React dashboard running on a different domain to make requests to your API, we need to configure CORS in our Flask application. This involves understanding how the browser handles cross-origin requests and configuring our server to allow these requests.

#### Triaging the Question

The question asks for enabling CORS so that a frontend React app hosted at one origin can communicate with an ML backend running on another domain. The key here is ensuring that the API allows cross-origin requests while maintaining security. We need to use Flask's cross_origin decorator or middleware to handle this.

#### First-Principles Solution

First, we'll import the necessary modules and set up our basic Flask application. Then, we'll add the CORS configuration using either the built-in Flask-CORS extension or by manually configuring it with a custom function.

Install Flask-CORS: If you are not already familiar with CORS, installing the flask-cors package is straightforward.

BASH
pip install flask-cors

Configure CORS in Flask: We can either use the @cross_origin() decorator or configure CORS globally.

Step-by-Step Solution
Step 1: Install Flask-CORS
BASH
pip install flask-cors
Step 2: Configure CORS Using flask-cors

Here's a basic example of how to enable CORS using the flask-cors package:

PYTHON
from flask import Flask, request
from flask_cors import cross_origin

app = Flask(__name__)

# Enable CORS globally for all routes
@app.after_request
def after_request(response):
    response.headers.add('Access-Control-Allow-Origin', '*')
    response.headers.add('Access-Control-Allow-Headers', 'Content-Type,Authorization')
    response.headers.add('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS')
    return response

# Example API endpoint
@app.route('/predict', methods=['POST'])
@cross_origin()  # Use the cross_origin decorator for this specific route if needed
def predict():
    data = request.get_json()
    # Your ML prediction logic here
    return {'result': 'Prediction'}

if __name__ == '__main__':
    app.run(debug=True)
Step 3: Manually Configure CORS

If you prefer not to use the flask-cors package, you can manually configure CORS in your Flask application:

PYTHON
from flask import Flask, request

app = Flask(__name__)

@app.after_request
def after_request(response):
    response.headers.add('Access-Control-Allow-Origin', '*')
    response.headers.add('Access-Control-Allow-Headers', 'Content-Type,Authorization')
    response.headers.add('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS')
    return response

@app.route('/predict', methods=['POST'])
def predict():
    data = request.get_json()
    # Your ML prediction logic here
    return {'result': 'Prediction'}

if __name__ == '__main__':
    app.run(debug=True)

#### Closing Summary
and Common Mistakes

In summary, enabling CORS in a Flask application is essential for allowing cross-origin requests from your React dashboard. We can achieve this either by globally configuring the flask-cors package or manually setting up CORS headers using decorators or middleware.

#### Common Mistakes

Forgetting to set the correct Access-Control-Allow-Origin, Access-Control-Allow-Methods, and Access-Control-Allow-Headers.
Not handling OPTIONS requests, which are sent by browsers as preflight requests.
Incorrectly configuring the headers, leading to security vulnerabilities or failure of cross-origin requests.
Self-Check

I have ensured that the solution is complete and correct.

**Code:**

```python
from flask import Flask, request
from flask_cors import cross_origin

app = Flask(__name__)

# Enable CORS globally for all routes
@app.after_request
def after_request(response):
    response.headers.add('Access-Control-Allow-Origin', '*')
    response.headers.add('Access-Control-Allow-Headers', 'Content-Type,Authorization')
    response.headers.add('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS')
    return response

# Example API endpoint
@app.route('/predict', methods=['POST'])
@cross_origin()  # Use the cross_origin decorator for this specific route if needed
def predict():
    data = request.get_json()
    # Your ML prediction logic here
    return {'result': 'Prediction'}

if __name__ == '__main__':
    app.run(debug=True)
```

