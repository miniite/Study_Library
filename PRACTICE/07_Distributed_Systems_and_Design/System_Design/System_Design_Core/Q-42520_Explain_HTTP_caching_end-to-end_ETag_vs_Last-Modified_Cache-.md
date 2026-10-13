---
type: "practice"
difficulty: "Deep"
---

# Explain HTTP caching end-to-end: ETag vs Last-Modified, Cache-Control max-age vs s-maxage, and stale-while-revalidate. How would you cache a personalized API response safely at a CDN?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Read-through and Write-behind Caching](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Read-through_and_Write-behind_Caching/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

HTTP caching is a fundamental concept in web performance optimization that leverages cached copies of responses to reduce network latency and bandwidth usage. To understand HTTP caching end-to-end, we need to dive into key headers like ETag, Last-Modified, Cache-Control max-age, s-maxage, and stale-while-revalidate. These headers work together to ensure efficient and safe cache management.

The question is probing your understanding of these mechanisms, the trade-offs involved, and how they interplay in a real-world scenario—specifically, caching personalized API responses at a Content Delivery Network (CDN). The core idea here is to balance between minimizing latency, ensuring data freshness, and maintaining data security.

#### Assumptions

The API response includes sensitive user information.
We aim to cache responses efficiently while respecting the privacy of users.
We are dealing with both HTTP/1.1 and HTTP/2 environments.
The CDN supports ETag, Last-Modified, Cache-Control, etc., headers.
Step-by-Step Solution
1. Understanding ETag vs. Last-Modified
ETag: A unique identifier that changes whenever the resource is modified.
Last-Modified: Indicates when a resource was last modified. It’s simpler and less secure than ETag because it doesn't uniquely identify different versions of the resource.

Both headers help in conditional GET requests, where the client can ask if a cached response is still valid by revalidating with the server using these headers.

2. Cache-Control max-age vs. s-maxage
max-age: Controls how long the cache entry should be considered fresh.
s-maxage: Similar to max-age but applies only to shared caches like CDNs, not user agents (browsers).

Using both in combination helps ensure that a CDN can serve stale content while still respecting browser caching policies.

3. stale-while-revalidate

This header instructs the client to use a cached response even if it’s older than its freshness lifetime and then revalidate with the server. This technique ensures good performance by serving a potentially outdated version of the resource but quickly refreshing it from the server side.

#### Example Scenario
Caching Personalized API Responses Safely

Resource Identification: Assume your API endpoint is /api/profile, and this response includes personalized data for each user.

ETag or Last-Modified: Use ETag to uniquely identify responses. This way, even if the response changes (e.g., due to an update in a user's profile), you can invalidate the cache appropriately.

Cache-Control Headers:

Set Cache-Control: max-age=3600 for controlling how long a browser should consider the cached response fresh.
Use s-maxage=1800 to allow CDNs to cache the response for 1800 seconds, but still respect the browser’s cache policy.

stale-while-revalidate: Enable this header on your server to instruct clients and CDNs to use stale content while revalidating with the server. This is crucial because:

It reduces the likelihood of serving stale data.
It allows for quick refreshes from the server without causing user experience disruptions.
5. Implementation Considerations
Privacy: Ensure that any personalized data included in API responses are encrypted and treated as sensitive information, even when cached.
Content Security Policies (CSP): Use CSP to prevent cached content from being executed or modified on other domains.
HTTP/2 Push: Leverage HTTP/2 push to pre-cache related resources, reducing the number of requests required for subsequent page loads.

#### Closing Summary

In summary, effectively caching personalized API responses at a CDN involves using ETag and Cache-Control headers to manage freshness and security. The stale-while-revalidate technique ensures that users get the latest data while keeping performance high. Proper implementation requires careful consideration of privacy, content security policies, and leveraging modern HTTP features.

#### Common Mistakes

Overlooking the importance of ETag in identifying unique responses.
Failing to set appropriate cache times for both browsers and CDNs.
Not understanding the role of stale-while-revalidate in maintaining a balance between performance and freshness.

**Comparison:**

| Header       | Description                                                                                             |
|--------------|---------------------------------------------------------------------------------------------------------|
| Cache-Control| Controls how long the cache entry should be considered fresh, and applies to both browsers and CDNs.  |
| ETag         | A unique identifier that changes whenever the resource is modified, used for conditional GET requests.   |
| s-maxage     | Specifies max-age only for shared caches (CDNs).                                                         |
| stale-while-revalidate | Instructs clients to use a cached response even if it's older than its freshness lifetime and then revalidate with the server.|

**Code:**

```python
# Example Cache-Control headers configuration
response.headers['Cache-Control'] = 'public, max-age=3600, s-maxage=1800'
response.headers['ETag'] = f'"{etag_value}"'
```

**Example input / output:**

```text
# Example input -> output for a GET request to /api/profile
GET /api/profile HTTP/1.1
Host: example.com

HTTP/1.1 200 OK
Cache-Control: public, max-age=3600, s-maxage=1800
ETag: "etag_value"
stale-while-revalidate: 60
Content-Type: application/json
Content-Length: 1234

{"user_id": "123", "name": "John Doe", "email": "john@example.com"}
```

**Data:**

```json
{
  "headers": {
    "Cache-Control": "public, max-age=3600, s-maxage=1800",
    "ETag": "\"etag_value\"",
    "stale-while-revalidate": "60"
  }
}
```

**Flow:**

1. Client makes GET request
2. Check cache
3. Cache hit, serve from cache
4. Cache miss, fetch from server
5. Server responds with ETag andCache-Control headers
6. Client stores response in cache
7. Client requests resource again
8. Check cache
9. Cache hit, serve from cache
10. Check for stale content usingETag
11. Serve from cache, revalidatewith server

