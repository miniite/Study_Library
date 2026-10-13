---
type: "practice"
difficulty: "Intermediate"
---

# How would you restrict CORS so only your frontend origin can call the Flask API, not all origins?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When securing a Flask API, restricting Cross-Origin Resource Sharing (CORS) is crucial to ensure that only requests from specific origins are allowed. For an intermediate engineer working on backend development with Flask, understanding and implementing this restriction properly is key. In this case, we want to allow only the frontend origin to make calls to our API while blocking all other origins.

#### Triage

The question asks us to restrict CORS in a Flask application so that it accepts requests only from a specific frontend domain (origin). This involves setting up the CORS configuration in our Flask app. The interviewer is likely probing your understanding of CORS, the Flask framework, and how to implement security measures.

#### Assumptions
We have a Flask backend running.
Our frontend application is hosted on a specific domain (e.g., http://example.com).
We want to restrict access to our API such that only requests from http://example.com are allowed.

#### First-Principles Solution

To restrict CORS in a Flask app, we need to configure the CORS extension. Here’s how we can do it:

Install and Import: If not already installed, install the flask-cors package via pip:

SH
pip install flask-cors

Configure CORS: In your Flask app, configure the CORS extension to allow requests only from a specific origin.

Here’s a step-by-step implementation:

PYTHON
from flask import Flask
from flask_cors import CORS

app = Flask(__name__)

# Configure CORS with allowed origins
cors = CORS(app, resources={r"/api/*": {"origins": "http://example.com"}})

@app.route('/api/data')
def get_data():
    return {'message': 'Hello from the API'}

if __name__ == '__main__':
    app.run(debug=True)
Testing: Test your Flask application to ensure that only requests from http://example.com are allowed and all other origins are blocked.

#### Closing Summary

By configuring the CORS extension in our Flask app, we can restrict access to our API so that it accepts requests only from a specific origin (in this case, http://example.com). This helps prevent unauthorized access and enhances the security of our backend service. Common mistakes include not setting up CORS correctly or allowing too many origins.

#### Common Mistakes/Follow-Ups
Allowing All Origins: Not specifying any allowed origins can be a significant security risk.
Inconsistent Testing: Failing to thoroughly test the CORS configuration might allow unwanted access.
Overly Broad Configuration: Sometimes, overly broad configurations can unintentionally open up the API to more origins than intended.

To further secure your application, consider additional measures such as rate limiting and authentication tokens.

