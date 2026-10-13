---
type: "practice"
difficulty: "Deep"
---

# What is CSRF, and why are token-based REST APIs called with an Authorization header less susceptible to it than cookie-based ones?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CORS](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Cross-Site Request Forgery (CSRF) is a security vulnerability that tricks users into submitting requests from one web site while the user's browser session with another web site is active. Token-based REST APIs, particularly those using Authorization headers, are less susceptible to CSRF attacks compared to cookie-based ones because they do not rely on cookies stored in the browser, which can be more easily manipulated by third-party sites.

#### Triaging the Question
The question asks about CSRF and contrasts token-based vs. cookie-based REST APIs with respect to their susceptibility to this type of attack. We need to explain what CSRF is, how it works, and why using Authorization headers makes token-based APIs less vulnerable compared to those that use cookies.

Step-by-Step Solution:

Understanding CSRF:

Definition: CSRF occurs when an attacker uses a victim's authenticated session to send unwanted requests to a web application.
How it works: An attacker crafts a malicious site or script that, when loaded in the user’s browser, sends a request to a target website with the user's session cookies attached. If the target site accepts this request based on the presence of valid cookies alone, the request is processed as if coming from the legitimate user.

Cookie-Based REST APIs:

Security Risks: In cookie-based systems, the Set-Cookie header in the server response sets a cookie that persists across browser sessions.
Vulnerability: If an attacker can trick the victim into loading their malicious site while they are authenticated on another site, the attacker's scripts can access and use the victim’s cookies to make unauthorized requests.

Token-Based REST APIs:

Security Principle: Token-based authentication typically involves obtaining a token (such as an Authorization header with a Bearer token) from the server.
Header Mechanism: Once obtained, this token is included in every API request via the Authorization header.
No Cookie Dependency: Unlike cookies, tokens are not stored persistently on the user’s browser. They are stateless and can be discarded after a single use.

Why Token-Based APIs Are Less Susceptible:

Browser Context Independence: Tokens do not rely on browser sessions or cookies to maintain authentication status. Therefore, even if an attacker can inject their scripts into a page viewed by the victim, they cannot easily obtain and reuse these tokens as they are often short-lived and specific to each request.
Stateless Nature: Token-based APIs operate in a stateless manner. Each request is independent of others, making it harder for attackers to exploit session data without additional authentication mechanisms.

#### Common Mistakes
and Follow-Ups:

Overreliance on Cookies: Failing to understand the importance of non-persistent storage for tokens can lead developers to underestimate the risk of CSRF in cookie-based systems.
Lack of Token Expiration: Not implementing token expiration or revalidation can make token-based APIs more vulnerable if the token is not refreshed regularly.
Cross-Origin Request Policies: Ensuring proper implementation of CORS policies to prevent cross-origin requests from unauthorized sources.

#### Closing Summary
Token-based REST APIs, especially those utilizing Authorization headers, are inherently less susceptible to CSRF attacks due to their stateless and non-persistent nature. This approach significantly reduces the risk compared to cookie-based systems where cookies can be easily manipulated by malicious scripts.

#### Common Mistakes

Overlooking the importance of token expiration and revalidation.
Misunderstanding how browser contexts handle tokens versus cookies.

#### Follow-Up
Questions:

How would you implement CSRF protection in a cookie-based API?
What are some best practices to mitigate CSRF attacks in web applications?

**Comparison:**

```
User loads malicious site
Malicious script sends requestto target API
API receives request withuser's session cookies
If cookie-based, process asauthenticated
If token-based, validate tokenfirst
Process request if valid
```

**Code:**

```python
# Example Python code for handling Bearer token authentication
from flask import Flask, request

app = Flask(__name__)

@app.route('/api/resource', methods=['GET'])
def protected_resource():
    # Check the Authorization header for a valid token
    auth_header = request.headers.get('Authorization')
    if not auth_header or 'Bearer ' not in auth_header:
        return "Unauthorized", 401

    token = auth_header.split()[1]

    # Validate the token here (e.g., from a database)
    if validate_token(token):
        return "Resource accessed successfully"
    else:
        return "Invalid token", 403
```

**Example input / output:**

| Input | Output          |
|-------|-----------------|
| GET /api/resource HTTP/1.1\r\nHost: example.com\r\nAuthorization: Bearer <valid-token>\r\n|\| HTTP/1.1 200 OK\r\nContent-Type: application/json\r\n{"message": "Resource accessed successfully"}

**Data:**

```json
{
  "headers": {
    "Authorization": "Bearer <valid-token>"
  }
}
```

**Flow:**

1. User logs in
2. Obtain token
3. Token stored securely
4. Make protected API calls usingAuthorization header
5. Server validates token, notcookies
6. Access granted if valid

