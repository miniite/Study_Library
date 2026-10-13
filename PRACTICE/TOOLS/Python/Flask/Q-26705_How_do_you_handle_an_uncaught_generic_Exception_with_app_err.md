---
type: "practice"
difficulty: "Deep"
---

# How do you handle an uncaught generic Exception with @app.errorhandler(Exception) and still return JSON?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle uncaught exceptions in a Flask application, we can use the @app.errorhandler decorator to catch general exceptions and ensure that they are returned as JSON responses. This approach ensures that our API remains consistent regardless of the error type. Here's how you would implement this:

Firstly, it’s important to understand what an uncaught exception is: it's an exception that occurs in your application but isn't caught by any try-except block. By default, Flask will terminate the request and return a generic HTTP 500 Internal Server Error along with an HTML error page. However, if you want to send more detailed information back to the client (typically a frontend application), JSON is preferable.

In this context, @app.errorhandler(Exception) allows us to catch any unhandled exception and provide custom handling for it. We can then serialize the error details into a JSON response that includes relevant data such as an error message, status code, and timestamp.

To implement this:

Define the Error Handler: Create a function decorated with @app.errorhandler(Exception).
Return JSON Response: Within the handler, return a dictionary containing the necessary information.
Serialization: Use Flask's built-in functions or custom serializers to ensure the response is correctly formatted as JSON.

Let’s walk through this in more detail:

#### Triage
What the Question Is Asking

The question asks how to use @app.errorhandler(Exception) effectively to handle any uncaught exception and return a JSON response. The goal is not just to catch exceptions but also to ensure that they are handled gracefully, providing useful information back to the client.

Key Assumptions
We assume that the Flask application has already been set up with an appropriate structure.
We want to maintain consistency in our error handling by always returning JSON responses.
We expect common uncaught errors like KeyError, TypeError, or custom exceptions.

#### Solution
From First Principles

Setup the Error Handler:

PYTHON
from flask import jsonify, request

@app.errorhandler(Exception)
def handle_exception(e):
    # Log the error for debugging purposes
    app.logger.error(f'Exception occurred: {str(e)}')

    # Prepare a response dictionary with relevant information
    resp = {
        "error": str(e),
        "timestamp": datetime.now().isoformat(),
        "status_code": 500,
        "request_url": request.url,
        "method": request.method
    }

    # Return the JSON response
    return jsonify(resp), 500

Explanation:

The handle_exception function logs the error using Flask's logger.
A dictionary is constructed containing the error message, timestamp, status code (defaulting to 500 for internal server errors), and details about the request that triggered the exception.
This dictionary is then returned as a JSON response with an HTTP 500 status code.

#### Closing Summary
and Common Mistakes/Follow-Ups

By using @app.errorhandler(Exception), you ensure that any uncaught exceptions in your application are handled gracefully, always returning consistent JSON responses. However, there are some common pitfalls to avoid:

Insufficient Error Logging: Make sure to log the error details properly so they can be debugged.
Hardcoded Status Codes: In this example, we hardcode a 500 status code, but depending on the nature of the exception, you might want to return different codes (e.g., 400 for client-side errors).
Complex Error Responses: For more complex applications, consider adding additional fields or even nested structures in your error response.

#### Example
Follow-Up

If the interviewer asks about handling specific types of exceptions separately:

You can define multiple @app.errorhandler functions to handle different exception types with tailored responses. For example:
PYTHON
@app.errorhandler(KeyError)
def handle_key_error(e):
    resp = {
        "error": f"Key Error: {e}",
        "status_code": 400,
        # Additional error-specific fields
    }
    return jsonify(resp), 400

**Code:**

```python
from flask import Flask, request, jsonify
import datetime

app = Flask(__name__)

@app.errorhandler(Exception)
def handle_exception(e):
    app.logger.error(f'Exception occurred: {str(e)}')

    resp = {
        "error": str(e),
        "timestamp": datetime.now().isoformat(),
        "status_code": 500,
        "request_url": request.url,
        "method": request.method
    }

    return jsonify(resp), 500

@app.route('/')
def index():
    raise ValueError("Test exception")  # Simulate an error
    return "Hello, World!"

if __name__ == '__main__':
    app.run(debug=True)
```

