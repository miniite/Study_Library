---
type: "practice"
difficulty: "Deep"
---

# How would you retrieve the client's IP address from a Flask request behind a proxy?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrieve the client's IP address from a Flask request when it is behind a proxy, we need to handle the X-Forwarded-For header that proxies typically set. This involves understanding how Flask processes requests and setting up appropriate configurations.

#### Triage

The question asks us to extract the original client’s IP address in a scenario where the server is behind a reverse proxy (like Nginx or Apache). The key here is recognizing that standard request.environ['REMOTE_ADDR'] will return the IP of the proxy, not the actual client. We need to look at the X-Forwarded-For header which contains the client’s original IP.

#### First-Principles Solution
Understand Flask's Request Handling: A Flask application uses request objects to handle incoming HTTP requests. The request.environ dictionary provides access to all environment variables set by the server.
Check X-Forwarded-For Header: When behind a proxy, you should inspect the X-Forwarded-For header in request.headers.
Handle Edge Cases: If multiple proxies are involved, the header might contain multiple IP addresses separated by commas.

#### Implementation

Here is how you can retrieve and handle the client’s IP address in Flask:

PYTHON
from flask import request

def get_client_ip():
    """
    Retrieves the original client's IP address from the X-Forwarded-For header.
    If no such header exists, returns REMOTE_ADDR as a fallback.
    """
    x_forwarded_for = request.headers.get('X-Forwarded-For')
    if x_forwarded_for:
        # Split the IPs and take the first one
        ip_address = x_forwarded_for.split(',')[0]
    else:
        # Use REMOTE_ADDR as default
        ip_address = request.remote_addr

    return ip_address

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this solution, we use request.headers.get('X-Forwarded-For') to safely retrieve the client's IP address. If multiple proxies are involved and there are several IPs in the header, splitting by commas allows us to get the first one as a reliable choice.

#### Common mistakes
include:

Failing to check for x_forwarded_for, leading to incorrect logs or security vulnerabilities.
Incorrectly assuming that request.remote_addr will always contain the client's IP, which can lead to false positives in proxy setups.

#### Follow-ups
might involve discussing how this interacts with different types of proxies (Nginx vs. Apache), handling cases where no header is set, and best practices for logging client IPs securely.

