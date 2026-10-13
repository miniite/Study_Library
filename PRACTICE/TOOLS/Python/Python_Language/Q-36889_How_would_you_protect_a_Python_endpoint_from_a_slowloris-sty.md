---
type: "practice"
difficulty: "Deep"
---

# How would you protect a Python endpoint from a slowloris-style or large-body attack, and what belongs in the app versus the reverse proxy layer?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To protect a Python endpoint from attacks like slowloris (a type of DoS attack that floods a server with numerous HTTP requests) or large-body attacks, we need to consider both application-level and reverse proxy-level defenses. The key is understanding the differences between what can be effectively managed at each layer.

#### Triaging the Question

The question asks about protecting Python endpoints specifically from slowloris-style attacks and large-body attacks. The interviewer is probing our knowledge of attack vectors in web applications, best practices for security, and the appropriate responsibilities of different layers (application vs. reverse proxy).

We assume that we are dealing with a typical setup where there's a reverse proxy like Nginx or HAProxy in front of a Python application running on Gunicorn or uWSGI.

#### First-Principles Solution

Reverse Proxy Layer:

Rate Limiting: The reverse proxy should implement rate limiting to prevent too many connections from a single IP address.
Connection Timeouts and Idle Timeouts: Configure these to close idle connections quickly, preventing them from keeping the server busy indefinitely.
Body Size Limits: Set limits on the maximum request body size to avoid processing large payloads.

Application Layer:

Connection Pooling: Ensure that connections are properly closed when they're no longer needed.
Streaming Responses: Use streaming responses where applicable to reduce memory usage and improve performance.
Error Handling and Logging: Implement robust error handling and logging to monitor and respond to suspicious activity.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the reverse proxy should handle rate limiting, connection timeouts, and body size limits. The application itself should focus on proper connection management, streaming responses, and thorough error handling. By distributing these responsibilities effectively, we can significantly mitigate the risk of slowloris-style attacks and large-body attacks.

#### Common mistakes
include:

Not configuring a reverse proxy at all or setting it up improperly.
Overloading the application code with complex rate limiting logic instead of relying on the reverse proxy.
Neglecting to monitor logs and error messages, leading to undetected attacks.

#### Follow-ups
might include discussing specific configuration examples for Nginx or HAProxy, comparing different reverse proxies based on performance and ease of use, or diving deeper into connection management strategies in Python applications.

**Comparison:**

```
Reverse Proxy Layer	Application Layer
Rate Limiting	Connection Pooling
Idle Timeouts	Streaming Responses
Body Size Limits	Error Handling
```

**Code:**

```python
# Example of setting up rate limiting in Flask with Flask-Limiter
from flask import Flask
from flask_limiter import Limiter

app = Flask(__name__)
limiter = Limiter(app, key_func=get_remote_address)

@app.route('/')
@limiter.limit("100/minute")
def index():
    return "Hello, World!"
```

**Example input / output:**

| Reverse Proxy Configuration | Description |
|-----------------------------|-------------|
| Nginx Rate Limiting          | `limit_req_zone $binary_remote_addr zone=my_limit:10m rate=1r/m;` and `limit_req zone=my_limit burst=10 nodelay;` |
| HAProxy Rate Limiting        | `tcp-request rate-limit 50 burst 10 deny 429` to limit requests per IP address |

**Data:**

```json
{
  "reverse_proxy": {
    "nginx": {
      "rate_limiting": {
        "limits": {"100/minute": "1.2.3.4"},
        "timeout_idle": 60,
        "max_body_size": 5242880
      },
      "configuration": """
       http {
           limit_req_zone $binary_remote_addr zone=my_limit:10m rate=1r/m;
           server {
               location / {
                   limit_req zone=my_limit burst=10 nodelay;
                   proxy_pass http://backend;
               }
           }
       }
      """
    },
    "haproxy": {
      "rate_limiting": {
        "limits": {"50/minute": "4.3.2.1"},
        "timeout_connect": 5s,
        "max_body_size": 8388608
      },
      "configuration": """
       backend app_servers
           balance roundrobin
           server web1 192.0.2.1:80 check
           server web2 192.0.2.2:80 check

       frontend http_front
           acl slowloris src 4.3.2.1
           tcp-request rate-limit 50 burst 10 deny 429
           default_backend app_servers
      """
    }
  },
  "application": {
    "gunicorn": {
      "settings": {
        "timeout": 60,
        "keepalive": 30
      }
    }
  }
}
```

**Flow:**

1. Reverse Proxy
2. Ratelimit
3. Idle Timeout
4. Body Size Limit
5. Application
6. Connection Pooling
7. Streaming Responses
8. Error Handling and Logging

