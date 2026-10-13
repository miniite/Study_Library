---
type: "practice"
difficulty: "Deep"
---

# What are g and current_app in Flask and when would you use them?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Flask, g and current_app are powerful tools that provide access to global data and context within a request lifecycle. Let's dive into what they mean, how they work, and when you might want to use them.

g is a global object that is unique per thread in the Flask application. It’s often used for storing temporary values such as user information or database connections during the processing of an HTTP request.

current_app, on the other hand, refers to the currently running application context. This can be useful when you need access to parts of the app state that aren’t directly available through g.

Here's a step-by-step breakdown and reasoning from first principles:

TL;DR
g: Used for storing per-request data.
current_app: Provides access to the current application context.

#### Triaging the Question

The question is asking about two specific attributes in Flask: g and current_app. The interviewer might be probing your understanding of request-scoped data storage, thread safety, and application context management.

#### Assumptions
You are familiar with multi-threaded environments.
You understand the importance of state management within web applications.

#### Solution

g Object:

g is a global object that gets reset at the start of each request. It's used for storing per-request data.

#### Example
use cases include:
Storing user information once authenticated.
Keeping track of database connections to avoid connection leaks.

current_app Object:

current_app is a proxy that refers to the currently running application’s context. This can be useful for accessing parts of the app state that aren't directly available through g.

#### Example
use cases include:
Retrieving configuration settings or environment-specific data.
Accessing global functions or decorators provided by your Flask application.
Step-by-Step Reasoning

Understanding g:

PYTHON
from flask import g

# Example usage: storing user information in a request
@app.before_request
def store_user_data():
    g.user = current_user  # Assuming you have an authenticated user stored somewhere

# In some route handler, we can access this user data as follows:
@app.route('/user')
def user_page():
    return f"Hello {g.user.username}"

Understanding current_app:

PYTHON
from flask import current_app

# Example usage: retrieving configuration settings
@app.route('/config')
def config_data():
    db_config = current_app.config['DATABASE_URI']
    return f"Database URI: {db_config}"

#### Closing Summary
and Common Mistakes/Follow-Ups

g is a powerful tool for storing per-request data, ensuring that each request has its own isolated environment. current_app, on the other hand, provides access to the current application’s context, allowing you to retrieve global settings or functions.

#### Common mistakes
include:

Not resetting g at the start of each request.
Misusing current_app when g is sufficient for storing per-request data.

#### Follow-up
questions might explore edge cases like handling concurrent requests and ensuring thread safety, as well as best practices for state management in Flask applications.

