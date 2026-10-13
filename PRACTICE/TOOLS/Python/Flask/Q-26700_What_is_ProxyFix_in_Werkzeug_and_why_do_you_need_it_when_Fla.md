---
type: "practice"
difficulty: "Deep"
---

# What is ProxyFix in Werkzeug and why do you need it when Flask sits behind Nginx or a load balancer?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ProxyFix in Werkzeug is an essential tool to configure Flask applications correctly when they are deployed behind a reverse proxy like Nginx or a load balancer. The primary role of ProxyFix is to adjust the WSGI environment dictionary passed from the reverse proxy to ensure that Flask has accurate information about the original client's request, such as headers and paths.

#### Triaging the Question

The question revolves around understanding why we need ProxyFix when using Flask with a reverse proxy like Nginx. The key aspects the interviewer is probing include:

Understanding of WSGI Environment: How the WSGI environment works in relation to HTTP requests.
Reverse Proxies and Load Balancers: Their role in handling client requests before they reach your application server.
Flask's Request Handling: How Flask interprets and uses information from the WSGI environment.

#### First Principles Solution

Flask, like many web frameworks, relies on the WSGI protocol to handle incoming HTTP requests. The WSGI environment dictionary contains all the necessary details about the request, such as headers, paths, query strings, etc., which are crucial for routing and processing the request correctly. However, when Flask is deployed behind a reverse proxy like Nginx or a load balancer, these details can be misinterpreted because the reverse proxy acts as an intermediary.

Here’s what happens:

Request from Client: A client sends a request to your server.
Reverse Proxy Involvement: The request first hits the reverse proxy (e.g., Nginx), which then forwards this request to the actual Flask application.
WSGI Environment Mismatch: By default, the WSGI environment dictionary contains headers and paths that point to the reverse proxy itself rather than the client.
Why You Need ProxyFix

To address this issue, you need to use ProxyFix. This middleware helps in:

Correcting Headers: Adjusts REMOTE_ADDR (client's IP) and other relevant headers to reflect the actual client.
Modifying Paths and Query Strings: Ensures that paths and query strings are correctly set as if they came directly from the client.

#### Implementation
Example

Here’s how you can configure ProxyFix in your Flask application:

PYTHON
from flask import Flask, request
from werkzeug.middleware.proxy_fix import ProxyFix

app = Flask(__name__)

# Configure ProxyFix to handle reverse proxies
app.wsgi_app = ProxyFix(app.wsgi_app)

@app.route('/')
def hello_world():
    return f"Hello, {request.remote_addr}!"

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, using ProxyFix is essential when deploying Flask applications behind a reverse proxy to ensure that the application receives accurate information about client requests. Without it, issues like incorrect IP addresses, malformed paths, or other WSGI environment inconsistencies can arise.

#### Common mistakes
include:

Forgetting to use ProxyFix: This leads to incorrect routing and processing of requests.
Misconfiguring the number of proxies: By default, ProxyFix assumes one reverse proxy. If there are multiple layers, you need to specify this with the proxies parameter.
Ignoring security implications: Ensure that IP addresses are correctly handled to prevent security risks like IP spoofing.
Common Follow-Ups
How many proxies do you think can handle if your setup has more than one?
Can ProxyFix also be used for modifying other parts of the WSGI environment, such as cookies or headers?
Are there any performance considerations when using ProxyFix?

