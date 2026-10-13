---
type: "practice"
difficulty: "Deep"
---

# How do you design a REST endpoint's response so a client can safely cache it locally and reconcile with a delta-sync endpoint, avoiding a full refetch when only a few records changed?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to design a REST API that ensures safe caching on the client side while enabling efficient reconciliation with a delta-sync endpoint. The key idea is to use a combination of cache-control headers and ETags, ensuring that the client can detect changes and only fetch updated records when necessary.

#### Triage

The core challenge here is to implement a mechanism that allows the client to safely cache responses, but also provides an efficient way for clients to update their local caches with minimal network requests. The client should be able to determine if the data has changed without performing a full refetch. We will explore how to achieve this using HTTP headers and JSON payloads.

#### First Principles Solution
Cache-Control Headers: Use these headers to instruct the client on how long it can safely cache the response.
ETag for Versioning: Generate an ETag based on the data's hash or a unique identifier, which the server can use to verify if the cached content is up-to-date.
Delta-Sync Endpoint: Implement an endpoint that provides only the changed records since the last sync.
Step-by-Step Solution
1. Cache-Control Headers

The client should be instructed to cache responses with a short duration, typically using max-age or s-maxage. For example:

HTTP
Cache-Control: max-age=3600

This instructs the client to cache the response for one hour.

2. ETag for Versioning

Generate an ETag based on the data's hash or a unique identifier. This can be done server-side using the following example:

PYTHON
import hashlib

def generate_etag(data):
    return hashlib.md5(data.encode()).hexdigest()

When returning the response, include the ETag in the ETag header:

HTTP
ETag: "abcdef123456"
3. Delta-Sync Endpoint

Implement a delta-sync endpoint that returns only the records that have changed since the last sync. The client should provide an If-None-Match header with its current ETag to request the delta:

HTTP
GET /delta-sync HTTP/1.1
Host: example.com
If-None-Match: "abcdef123456"

The server then checks if the provided ETag matches the current one and returns only the changed records:

HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "changedRecords": [
        {"id": 1, "name": "Updated Name", "lastModified": "2023-10-10T14:00:00Z"}
    ]
}

If there are no changes, the server returns a 304 Not Modified response.

#### Closing Summary

By using cache-control headers and ETags, we ensure that clients can safely cache responses. The delta-sync endpoint provides an efficient way to update local caches with minimal network requests, thus avoiding full refetches when only a few records have changed.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to use the ETag header in responses and checks.
Mistake: Not handling the 304 Not Modified response properly on the client side.

#### Follow-Up
Discussing how to handle concurrent updates and ensuring data consistency.

**Comparison:**

```
Cache-Control	ETag Usage
max-age=3600	For caching
If-None-Match	For delta-sync
```

**Code:**

```python
import hashlib

def generate_etag(data):
    return hashlib.md5(data.encode()).hexdigest()
```

**Example input / output:**

```

Client Request:

HTTP
Copy
GET /data HTTP/1.1
Host: example.com
Cache-Control: max-age=3600
If-None-Match: "abcdef123456"

Server Response (with delta sync):

HTTP
Copy
HTTP/1.1 200 OK
Content-Type: application/json

{
    "changedRecords": [
        {"id": 1, "name": "Updated Name", "lastModified": "2023-10-10T14:00:00Z"}
    ]
}

Server Response (no changes):

HTTP
Copy
HTTP/1.1 304 Not Modified
ETag: "abcdef123456"
```

**Data:**

```json
{
    "ETag": "abcdef123456"
}
```

**Flow:**

1. No
2. Yes
3. Client Requests Data
4. Server Returns Data with ETag
5. ETag Matches?
6. Fetch Delta Sync
7. Cache Hit, Use Local Copy

