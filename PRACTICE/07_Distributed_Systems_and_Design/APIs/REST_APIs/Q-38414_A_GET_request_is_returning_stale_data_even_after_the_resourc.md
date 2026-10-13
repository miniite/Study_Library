---
type: "practice"
difficulty: "Deep"
---

# A GET request is returning stale data even after the resource was updated. How would you investigate?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a GET request returns stale data, it's crucial to understand that this issue can stem from various sources, including client-side caching mechanisms, server-side configurations, and network delays. Our goal is to systematically identify the root cause and address it. Let’s break down the problem step by step.

#### Triage

The question asks us to investigate why a GET request might return stale data even after the resource has been updated. The interviewer is probing our understanding of caching mechanisms, REST principles, and server configurations. We need to consider both client-side (browser) and server-side (application and network) factors that could be contributing to this issue.

#### Assumptions
The resource in question is a standard web resource with an associated URL.
There are no external systems or third-party services involved that might affect the request response.
We have access to logs, server monitoring tools, and the ability to manipulate headers and responses.
The client's browser cache might be involved.

#### First-Principles Solution

Client-Side Caching:

Check if the client (browser) is caching the data. This can happen due to HTTP caching mechanisms or browser settings. To investigate, we can perform a hard refresh (Ctrl+Shift+R on most browsers) and see if the issue persists.

Server-Side Response Headers:

Verify the server response headers for Cache-Control, Expires, and ETag. These headers control how long the resource is considered fresh by the client.
HTTP
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0

ETag and Conditional Requests:

Ensure that ETags are being used to perform conditional GET requests. If the server responds with a 200 OK without an ETag, it might indicate that the client is not handling this correctly.
HTTP
ETag: "unique-identifier"

Network and DNS Issues:

Check for any network or DNS issues that might be causing the client to fetch a cached version of the resource instead of the updated one.

Server Configuration:

Review the server configuration to ensure it’s sending the correct headers and is not inadvertently caching responses.
PYTHON
from flask import make_response

@app.route('/resource')
def get_resource():
    response = make_response(render_template('template.html'))
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['Expires'] = '0'
    return response

Database and Cache Consistency:

Ensure that the database is updated correctly and any caching mechanisms (e.g., Redis) are synchronized with the database updates.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To summarize, we have systematically investigated multiple potential causes of stale data in a GET request:

Client-side caching mechanisms.
Incorrect server response headers.
Network and DNS issues.
Server configuration errors.
Database and cache consistency issues.

#### Common mistakes
include overlooking the client's role in caching or assuming that only the server is responsible for freshness. Follow-ups could involve detailed logs analysis, specific network packet captures (using tools like Wireshark), and testing with different browsers to isolate the issue.

**Comparison:**

```
Header	Value
Cache-Control	no-cache, no-store, must-revalidate
Pragma	no-cache
Expires	0
```

**Code:**

```python
from flask import make_response

@app.route('/resource')
def get_resource():
    response = make_response(render_template('template.html'))
    response.headers['Cache-Control'] = 'no-cache, no-store, must-revalidate'
    response.headers['Pragma'] = 'no-cache'
    response.headers['Expires'] = '0'
    return response
```

**Example input / output:**

```
Input: GET request to /resource with cache headers.
Output: Response headers with Cache-Control, Pragma, and Expires set appropriately.
HTTP
Copy
GET /resource HTTP/1.1
Host: example.com
Cache-Control: max-age=0, no-cache, must-revalidate, proxy-revalidate

Response:

HTTP
Copy
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 3456
Cache-Control: no-cache, no-store, must-revalidate
Pragma: no-cache
Expires: 0

<!DOCTYPE html>
<html>
<head>
    <title>Resource Page</title>
</head>
<body>
    <!-- Content -->
</body>
</html>
```

**Data:**

```json
{
  "headers": {
    "Cache-Control": "no-cache, no-store, must-revalidate",
    "Pragma": "no-cache",
    "Expires": "0"
  }
}
```

**Flow:**

1. Start
2. Check Client Caching
3. Hard Refresh the Browser
4. Check Cache-Control Headerson Server Response
5. Verify ETags and ConditionalRequests
6. Check Network/DNS Issues
7. Review Server Configuration
8. Database/Cache Consistency
9. End

