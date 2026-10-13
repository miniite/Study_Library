---
type: "practice"
difficulty: "Intermediate"
---

# What does @app.errorhandler do and how would you register a global error handler?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Error Handling](<../../../../TOOLS/Python/Flask/Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let's quickly define what @app.errorhandler is in Flask. It’s a decorator used to register error handlers with your application. This allows you to define functions that will be called when specific HTTP errors occur, enabling you to customize the response and user experience for those errors.

When an error occurs within a route in Flask, by default, it will raise an exception which is then handled by the framework to generate an appropriate error page. However, using @app.errorhandler, you can intercept these exceptions and provide more tailored responses.

#### Triage
What's Being Probed

The interviewer is probing your understanding of how error handling works in Flask applications, as well as your ability to write custom error handlers. This involves knowing the difference between default behavior and custom handling, and how to implement this effectively.

#### Assumptions
The candidate has a basic understanding of Flask.
We are working with a Flask application.
You understand HTTP status codes and their meanings.

#### First-Principles Solution

Let’s break down the process step by step:

Understanding @app.errorhandler:

This decorator is used to register a function that will be called when an error occurs in the specified HTTP error.
The argument to @app.errorhandler should be one or more HTTP status codes (e.g., 404, 500).

Registering Error Handlers:

You can define a function with a specific HTTP status code as an argument and use @app.errorhandler before it.
Inside the handler function, you can customize the response according to your needs.

Here's how you would register a global error handler:

PYTHON
from flask import Flask

app = Flask(__name__)

@app.errorhandler(404)
def not_found_error(error):
    return "This is a custom 404 error message", 404

@app.errorhandler(500)
def internal_server_error(error):
    return "An internal server error occurred", 500
Customizing Responses:
You can return different types of responses, such as simple strings or even render a template.
Customizing the response helps in providing better user experience and making your application more robust.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, @app.errorhandler is a powerful tool to handle specific HTTP errors in Flask. By registering custom error handlers, you can provide meaningful responses to users and improve the overall reliability of your application.

#### Common mistakes
include:

Not covering all important status codes.
Failing to return appropriate status codes with your response (e.g., 404 for not found, 500 for internal server errors).

#### Follow-ups
might involve discussing how these error handlers interact with Flask’s logging mechanisms or how you could use them in a production environment.

