---
type: "practice"
difficulty: "Basic"
---

# How do you define a route that accepts multiple HTTP methods like GET and POST?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Routing and Blueprints](<../../../../TOOLS/Python/Flask/Routing_and_Blueprints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To define a route in Flask that can accept both GET and POST requests, we need to understand how Flask handles routes and request methods. The core idea is to use the appropriate decorators provided by Flask to specify which HTTP methods are allowed for a given route.

Firstly, let's break down what this question is asking:

We're working with Flask, a micro web framework in Python.
We want to create a single route that can handle both GET and POST requests.
This involves understanding how routes and request handlers are defined in Flask.
Step-by-Step Solution

Import the necessary modules:

PYTHON
from flask import Flask, request

Initialize the Flask application:

PYTHON
app = Flask(__name__)

Define the route and specify allowed HTTP methods using decorators:

The @app.route decorator is used to bind a function to a URL.
By default, routes handle GET requests, but we can add other methods like POST.

Here’s an example:

PYTHON
@app.route('/example', methods=['GET', 'POST'])
def handle_request():
    if request.method == 'GET':
        return "This is a GET request"
    elif request.method == 'POST':
        # Process the POST data here
        return "This is a POST request", 201

#### Closing Summary
and Common Mistakes
Summary: In Flask, we define a route that accepts multiple HTTP methods by using the methods parameter in the @app.route decorator. This allows us to handle both GET and POST requests for the same URL.

#### Common Mistakes
Forgetting to specify the allowed methods in the @app.route decorator.
Not checking the request.method variable inside the route handler, which can lead to unexpected behavior if different HTTP methods are not properly handled.
Explanation

When a client sends an HTTP request to /example, Flask will call the handle_request function based on the method type:

If it’s a GET request, the response will be "This is a GET request".
If it’s a POST request, you can process the data as needed and return a 201 status code indicating successful creation.

By following these steps, we ensure that our Flask application can handle multiple HTTP methods for a single route efficiently.

