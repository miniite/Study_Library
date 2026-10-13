---
type: "practice"
difficulty: "Deep"
---

# How would you implement rate limiting on a Flask API?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1 · [↑ Long-running Work in Flask](<../../../../TOOLS/Python/Flask/Long-running_Work_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To implement rate limiting on a Flask API, we need to control the number of requests that can be made to an endpoint within a certain time frame. This is crucial for preventing abuse and ensuring fair usage of resources. At a high level, we will use middleware to inspect incoming requests and enforce the limits based on IP address or API keys.

#### Triage

The question probes our understanding of Flask's request handling mechanisms and how we can introduce non-trivial logic into our application flow without tightly coupling it with specific endpoints. We need to consider different strategies for rate limiting, such as using in-memory stores (e.g., Redis) or database-backed solutions. Additionally, the interviewer may be interested in trade-offs between performance and complexity.

#### Assumptions
Rate Limiting Mechanism: Use an in-memory store like Redis.
Rate Limit Configuration: Define a limit of X requests per minute for each unique client (IP or API key).
Granularity: We can use both IP addresses and API keys to enforce rate limits.

#### First-Principles Solution

Install Required Libraries:

BASH
pip install Flask Flask-Limiter redis

Set Up Redis Client:

PYTHON
from flask import Flask, request
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

# Configure rate limiting using Redis as the storage backend
limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

Define API Endpoints:

PYTHON
@app.route('/api/v1/resource')
@limiter.limit("10/minute")  # Limit to 10 requests per minute
def resource():
    return {"message": "Resource accessed"}

Handling Rate Limits Exceeded: By default, Flask-Limiter will return a HTTP 429 (Too Many Requests) response with an appropriate message. We can customize the error response if needed.

#### Closing Summary

Implementing rate limiting in Flask is straightforward with the help of libraries like Flask-Limiter. This approach allows us to easily control request rates without modifying existing endpoints significantly. Using Redis as a backend provides scalability and performance benefits, making it suitable for production environments.

#### Common Mistakes/Follow-Ups
Not Configuring Rate Limits Properly: Ensure you configure rate limits based on the expected usage pattern.
Over-Engineering: While it’s good to consider more advanced options like database-backed solutions, in many cases, Redis-based rate limiting is sufficient and performs well.
Ignoring API Keys/Headers: If using API keys, ensure they are properly configured as key_func parameter.

**Code:**

```python
from flask import Flask
from flask_limiter import Limiter
from flask_limiter.util import get_remote_address

app = Flask(__name__)

# Configure rate limiting using Redis as the storage backend
limiter = Limiter(
    app,
    key_func=get_remote_address,
    default_limits=["200 per day", "50 per hour"]
)

@app.route('/api/v1/resource')
@limiter.limit("10/minute")  # Limit to 10 requests per minute
def resource():
    return {"message": "Resource accessed"}

if __name__ == '__main__':
    app.run()
```

**Example input / output:**

```text
Input:
- Request sent every second to /api/v1/resource

Output:
- Within the first 60 seconds, 10 requests are processed successfully.
- After 61st request, a 429 Too Many Requests is returned.
```

