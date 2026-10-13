---
type: "practice"
difficulty: "Basic"
---

# How do you read a cookie sent by the client in a Flask request?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1 · [↑ Authentication in Flask](<../../../../TOOLS/Python/Flask/Authentication_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To read a cookie sent by the client in a Flask request, we need to understand that cookies are part of the HTTP headers sent with the client's request. In Flask, we can access these headers using the request object provided by Flask's built-in flask.request module.

TL;DR

In Flask, you use the request.cookies dictionary to read cookie values from the client's request. This is a simple and straightforward process that involves understanding HTTP requests and how cookies are structured in those requests.

#### Triaging the Question

The question is essentially asking for the method to extract cookie data from an incoming HTTP request within a Flask application. The interviewer might be probing your knowledge of:

HTTP Requests: Understanding how cookies are transmitted via headers.
Flask API: Familiarity with accessing specific parts of the request object.

#### First-Principles Solution

First, let's break down the HTTP request flow and then look at the Flask-specific approach to reading cookies.

HTTP Request Basics

When a client makes an HTTP request, it can include cookies in the headers. These cookies are typically sent as part of the Cookie header field. The server-side application (in this case, our Flask app) needs to parse these headers to extract and use the cookie values.

Flask's Approach

Flask provides a convenient way to access all HTTP headers through its built-in request object. Specifically, you can use request.cookies to get a dictionary of cookies sent by the client.

Here’s how you would do it in code:

PYTHON
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    # Accessing cookie values
    username = request.cookies.get('username')

    if username is None:
        return "No username cookie found"
    else:
        return f"Hello, {username}!"

In this example, request.cookies is a dictionary that contains all cookies sent by the client. The .get() method safely retrieves the value of 'username' from the cookies, returning None if no such cookie exists.

#### Closing Summary
and Common Mistakes

To summarize, reading a cookie in Flask involves using the request.cookies attribute to access a dictionary of cookies. This is a straightforward process but can be error-prone without understanding the basics of HTTP headers and the structure of a Flask request object.

#### Common mistakes
include:

Not checking if the cookie exists: Directly accessing request.cookies['username'] will raise a KeyError if 'username' does not exist, so always use .get() or similar methods to safely retrieve values.
Misunderstanding the HTTP headers: Cookies are part of the Cookie header, and you need to handle them as such.
Code Block
PYTHON
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    username = request.cookies.get('username')

    if username is None:
        return "No username cookie found"
    else:
        return f"Hello, {username}!"

**Code:**

```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def index():
    username = request.cookies.get('username')

    if username is None:
        return "No username cookie found"
    else:
        return f"Hello, {username}!"
```

