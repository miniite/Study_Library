---
type: "practice"
difficulty: "Intermediate"
---

# How do you build a Response object explicitly using make_response() and set a cookie on it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1 · [↑ Authentication in Flask](<../../../../TOOLS/Python/Flask/Authentication_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a Response object in Flask and set a cookie, you first need to understand that make_response() is used to create an explicit response from scratch or modify an existing one. This function allows you to manipulate the HTTP response before sending it back to the client. Here’s how you can do it:

Summary: You'll use flask.make_response() to construct a response and then set a cookie using the .set_cookie() method on that response object.

#### Triage
The question is asking for a practical example of setting up a Flask response with a custom cookie. The interviewer might be probing your understanding of HTTP responses, cookies, and how they are managed in Flask.

#### First-Principles Solution

Start by importing the necessary modules.
Create an empty response using make_response.
Use .set_cookie() to add a cookie to the response.
Return the modified response object from your route handler.

#### Closing Summary
+ Common Mistakes/Follow-Ups: This is a straightforward task, but ensure you handle edge cases and understand how cookies work in Flask.

PYTHON
from flask import make_response

@app.route('/set-cookie')
def set_cookie():
    # Create an empty response object
    response = make_response("Cookie has been set!")

    # Set a cookie with the name 'user_id' and value '12345'
    response.set_cookie('user_id', '12345')

    return response

