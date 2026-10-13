---
type: "practice"
difficulty: "Intermediate"
---

# What is an ETag and how does conditional caching with If-None-Match work?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Web and Browser Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Web_and_Browser_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with web caching, one critical aspect to understand is the ETag header, which plays a pivotal role in implementing efficient and conditional caching strategies. Specifically, ETags enable a server to provide fresh content or return an indication that it hasn’t changed since its last request, thus reducing unnecessary bandwidth usage and improving user experience.

To begin with, let's define what ETag is and how it works. An ETag is a unique identifier associated with each version of a resource (like a webpage). When a client requests a resource from the server, the server can include an ETag in the response header. This ETag acts as a checksum or fingerprint for the content.

The core idea behind conditional caching using If-None-Match is to check if the cached content on the client’s side is still valid before making another request to the server. Essentially, it allows the server to tell the client "You have the most recent version of this resource" without having to resend the entire content.

#### Triage

The question asks for an explanation of what ETag is and how conditional caching with If-None-Match works. The interviewer might be probing your understanding of HTTP caching mechanisms, the specifics of ETags, and how they integrate into a larger caching strategy.

#### Assumptions
We are discussing a scenario where resources (like web pages) are frequently accessed.
There is an existing cache mechanism in place, possibly on the client-side or within a CDN.
The server supports HTTP 1.1 and above, which includes ETag capabilities.

#### First-Principles Solution

The process starts with the client sending a request to the server for a resource. If the client has a previous version of this resource in its cache, it will include an If-None-Match header in the request. This header contains the ETag value from the last known cached version.

Client Request: The client requests a resource using a GET or similar HTTP method.
Server Response:
If the server finds that the ETag in the If-None-Match header matches the current ETag of the resource, it sends a 304 Not Modified response. This tells the client to use its cached version without downloading anything new.
If the ETags do not match, the server responds with the latest version of the resource along with an updated ETag. The client then updates its cache.

#### Example
Workflow

Initial Request:

PLAINTEXT
GET /path/to/resource HTTP/1.1
Host: example.com
Server response includes an ETag header:
HTTP
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
ETag: "abc123"

Subsequent Conditional Request (client has cached version with ETag: "def456"):

PLAINTEXT
GET /path/to/resource HTTP/1.1
Host: example.com
If-None-Match: "def456"
Server checks the ETag and since it does not match:
HTTP
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
ETag: "abc123"

#### Closing Summary
+ Common Mistakes/Follow-Ups

This method ensures that the server only sends data when necessary, saving bandwidth and reducing load times for users. It is particularly useful in scenarios where content might be frequently updated but not completely changed every time.

#### Common mistakes
include:

Not understanding that ETags are specific to each resource.
Failing to recognize the difference between ETag and other cache control mechanisms like Last-Modified.
Ignoring edge cases, such as handling preconditions in concurrent updates or dealing with different caching strategies across clients and servers.

#### Follow-up
questions might include:

How does ETag interact with other HTTP headers for caching?
What are the implications of using weak versus strong ETags?
How can we ensure that ETags remain unique and consistent over time?

**Comparison:**

```
Client Request	ETag Header Present?	Server Response
Initial GET	No	200 OK, ETag: "abc123"
Subsequent GET	Yes, If-None-Match	304 Not Modified
```

**Code:**

```python
# Example Python code to handle conditional GET requests with ETags
def handle_request(request):
    if 'If-None-Match' in request.headers:
        etag = request.headers['If-None-Match']
        # Check cache for the specific resource and its ETag
        cached_etag = get_cached_etag(resource)
        if cached_etag == etag:
            return 304, "Not Modified"
    # Serve fresh content
    response_body = serve_resource(resource)
    new_etag = calculate_etag(response_body)
    set_cache(resource, new_etag, response_body)
    return 200, response_body, {'ETag': new_etag}
```

**Example input / output:**

```

Initial Request:

PLAINTEXT
Copy
GET /path/to/resource HTTP/1.1
Host: example.com
Server Response:
HTTP
Copy
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
ETag: "abc123"

Subsequent Conditional Request (client has cached version with ETag: "def456"):

PLAINTEXT
Copy
GET /path/to/resource HTTP/1.1
Host: example.com
If-None-Match: "def456"
Server Response:
HTTP
Copy
HTTP/1.1 304 Not Modified
ETag: "abc123"
```

**Data:**

```json
{
  "resource_path": "/path/to/resource",
  "etag": "abc123"
}
```

**Flow:**

1. No
2. Yes
3. Client sends initial request
4. Server responds with contentand ETag
5. ETag match?
6. Client uses cached version
7. Server returns 304, clientupdates cache

