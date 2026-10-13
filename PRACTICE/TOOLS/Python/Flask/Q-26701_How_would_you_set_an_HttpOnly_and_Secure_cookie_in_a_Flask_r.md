---
type: "practice"
difficulty: "Deep"
---

# How would you set an HttpOnly and Secure cookie in a Flask response and why do those flags matter?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Authentication in Flask](<../../../../TOOLS/Python/Flask/Authentication_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Flask, setting an HttpOnly and Secure cookie ensures enhanced security by restricting client-side scripts from accessing the cookie's contents (via HttpOnly) and ensuring that the cookie is only sent over a secure HTTPS connection (via Secure). These flags help prevent common vulnerabilities such as Cross-Site Scripting (XSS) attacks.

#### Triage

The question asks how to set these specific flags on a cookie in Flask, which involves understanding HTTP headers. The interviewer is probing your knowledge of security practices and the capabilities provided by Flask. We'll assume that you're familiar with basic Flask setup and working within a Python environment where you can manipulate HTTP responses.

#### Solution
from First Principles

To set an HttpOnly and Secure cookie in a Flask response, you need to modify the set_cookie() method of a response object. This method takes several parameters, including the name and value of the cookie, as well as flags like httponly and secure.

Here’s how you can do it:

PYTHON
from flask import make_response

def set_secure_and_httponly_cookies():
    # Create a response object
    resp = make_response('Setting secure and HttpOnly cookies')

    # Set the cookie with HttpOnly and Secure flags
    resp.set_cookie(
        'session_id',
        value='random_value',
        httponly=True,
        secure=True,
        samesite='Lax'  # Optional: Define SameSite policy to mitigate CSRF attacks
    )

    return resp

# Example usage:
response = set_secure_and_httponly_cookies()

#### Closing Summary
and Common Mistakes/Follow-Ups

In the example above, we've demonstrated how to create a Flask response that includes an HttpOnly and Secure cookie. It's important to always use these flags where possible to enhance security, especially in environments that support HTTPS.

#### Common Mistakes
Forgetting to Use Secure Flag: If you deploy your application over HTTP (not HTTPS), the Secure flag might not be honored by the browser.
Not Using HttpOnly Properly: Failing to use the HttpOnly flag can leave your cookies vulnerable to XSS attacks.

#### Follow-Ups
Discuss scenarios where you might need to disable these flags, such as using JavaScript for cookie manipulation or testing in an insecure environment.
Explore other security-related aspects of Flask, like user session management and CSRF protection.

**Code:**

```python
from flask import make_response

def set_secure_and_httponly_cookies():
    resp = make_response('Setting secure and HttpOnly cookies')
    resp.set_cookie(
        'session_id',
        value='random_value',
        httponly=True,
        secure=True,
        samesite='Lax'
    )
    return resp
```

**Example input / output:**

```text
Input: A request to set a session cookie for the user.

Output: Flask response setting an HttpOnly and Secure cookie.
```

