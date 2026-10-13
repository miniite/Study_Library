---
type: "practice"
difficulty: "Basic"
---

# How do you run a Flask app in debug mode and what does debug mode give you?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To run a Flask application in debug mode, you would typically execute flask run --debugger-on from your terminal. Debug mode provides several benefits such as live-reloading of templates and static files, detailed error messages including stack traces, and automatic reloading when changes are made to the codebase.

The question is probing your understanding of the Flask framework's capabilities and the benefits it offers. It also checks if you know how to enable these features in a development environment.

#### Triage
Question: How do you run a Flask app in debug mode?
Probing Points:
Knowledge of basic Flask command-line interface (CLI) commands.
Understanding of what debug mode provides during development.
Familiarity with common best practices for local development.

#### First Principles Solution

Flask's debug mode is a feature that allows developers to test and develop their applications more effectively. When you run your application in debug mode, Flask automatically reloads the server when changes are made to your source code or templates. This means there’s no need to manually restart the server each time you make small adjustments.

Here’s how to start a Flask app in debug mode:

Installation and Setup: Ensure that Flask is installed in your Python environment. If not, you can install it via pip:

BASH
pip install flask

Create Your Application: Write your basic Flask application with routes and templates. For example, create a file named app.py:

PYTHON
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello World!"

if __name__ == '__main__':
    app.run(debug=True)

Running the App in Debug Mode: To run your application with debug mode enabled, simply execute:

BASH
python app.py

This command starts a local development server that listens on http://127.0.0.1:5000 by default.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, running your Flask application in debug mode is straightforward and enables faster development cycles. However, it’s important to disable this feature before deploying your app to production environments due to security concerns and performance overhead.

#### Common mistakes
include:

Forgetting to disable debug mode before deployment.
Running the server with elevated privileges while in debug mode (this can be a security risk).

#### Follow-up
questions might include discussing how to securely handle errors, especially when running in a live environment, or exploring more advanced configuration options available in Flask’s CLI.

**Code:**

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def home():
    return "Hello World!"

if __name__ == '__main__':
    app.run(debug=True)
```

