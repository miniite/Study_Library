---
type: "practice"
difficulty: "Deep"
---

# Explain how you would design negative caching for a 'resource not found' response so an attacker cannot flood you with junk keys and exhaust memory.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Caching Expensive Queries](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Expensive_Queries/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Negative caching involves storing "not found" responses in a cache to prevent repeated requests from overwhelming your backend. This strategy helps mitigate denial-of-service (DoS) attacks by limiting redundant queries, thus saving resources and ensuring the system remains responsive.

#### Triaging the Question

The core of this question is about designing a robust caching mechanism that specifically handles 'resource not found' responses in a way that doesn't flood the cache with junk keys. The interviewer is probing your understanding of caching strategies, DoS protection mechanisms, key management, and resource allocation trade-offs.

#### Assumptions
We are dealing with HTTP requests.
"Resource not found" typically translates to a 404 response.
The system has a finite amount of cache memory available.
Step-by-Step Solution:

Understanding the Problem:

Negative caching involves storing an entry in the cache when a resource is not found, indicating that future requests for this non-existent resource will also return a 'not found' response without hitting the backend.
This helps prevent attackers from flooding your system with repeated 404 requests, thereby reducing unnecessary load on your backend.

Designing the Cache Entry:

Key Generation: The cache key should be based on a unique identifier for each resource that is requested. For example, it could be derived from the URL or a combination of query parameters.
JAVASCRIPT

#### cons
t cacheKey = buildCacheKey(request.url, request.queryParams);
Negative Cache Entry Structure:
Key: cacheKey
Value: A structure that includes details about when the resource was last checked and whether it exists or not.
JSON
{
  "exists": false,
  "lastChecked": Date.now()
}

Cache Lookup and Insertion:

When a request is made, first check if the key already exists in the cache with exists set to false.
JAVASCRIPT

#### cons
t cachedEntry = cache.get(cacheKey);
If it does, return the 404 response directly from the cache.
JAVASCRIPT
if (cachedEntry && !cachedEntry.exists) {
  return new Response("Not Found", { status: 404 });
}

Handling Cache Miss:

If there's no cached entry or the exists field is true, proceed to check the backend.
After a successful 404 response from the backend, store this result in the cache for future requests.
JAVASCRIPT

#### cons
t backendResponse = await fetchFromBackend(request.url);
if (backendResponse.status === 404) {
  // Cache negative result with an expiry time to avoid false positives due to transient issues.

#### cons
t expirationTime = Date.now() + 60 * 1000; // Cache for 1 minute
  cache.set(cacheKey, { exists: false, lastChecked: expirationTime });
}

Expiring Cache Entries:

Ensure that cached negative responses have a finite life span to prevent long-term caching of false negatives.
Implement an expiry mechanism so entries get invalidated over time.

Rate Limiting and Defense in Depth:

To further protect against DoS attacks, implement rate limiting on both the frontend and backend levels.
JAVASCRIPT
if (rateLimitExceeded(request.ip)) {
  return new Response("Too Many Requests", { status: 429 });
}

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, negative caching for 'resource not found' responses is an effective strategy to mitigate DoS attacks and conserve backend resources. By carefully designing cache keys and handling expiration times, we can ensure that the system remains resilient and responsive.

#### Common Mistakes

Failing to set appropriate expiry times on cached entries.
Not implementing rate limiting, which can still be exploited by determined attackers.
Ignoring false positives in the cache, leading to unnecessary backend requests.

#### Follow-Ups

How would you handle different types of 404 responses (e.g., temporary vs. permanent)?
What are the implications of caching on data consistency?
How would you scale this solution for a high-traffic application?

**Comparison:**

```
Cache Key	Value
https://example.com/resource?key=value	{ exists: false, lastChecked: 1635879200000 }
```

**Code:**

```javascript
function buildCacheKey(url, queryParams) {
  return `${url}?${new URLSearchParams(queryParams).toString()}`;
}

async function fetchFromBackend(url) {
  // Simulate backend request.
  const response = await fetch(url);
  if (response.status === 404) {
    throw new Error("Resource not found");
  }
  return response;
}

function cacheKeyExists(cache, key) {
  return cache.get(key) !== null;
}

async function handleRequest(request, cache) {
  const cacheKey = buildCacheKey(request.url, request.queryParams);

  // Check if the entry exists and indicates a negative result.
  const cachedEntry = cacheKeyExists(cache, cacheKey)
    ? cache.get(cacheKey)
    : null;

  if (cachedEntry && !cachedEntry.exists) {
    return new Response("Not Found", { status: 404 });
  }

  // Proceed to backend check.
  try {
    const backendResponse = await fetchFromBackend(request.url);
    if (backendResponse.status === 404) {
      cache.set(cacheKey, { exists: false, lastChecked: Date.now() + 60 * 1000 });
    }
    return backendResponse;
  } catch (error) {
    // Handle any exceptions and ensure a consistent response.
    if (!cachedEntry || !cachedEntry.exists) {
      cache.set(cacheKey, { exists: false, lastChecked: Date.now() + 60 * 1000 });
    }
    return new Response("Not Found", { status: 404 });
  }
}
```

**Example input / output:**

```text
const request = new Request("https://example.com/resource?key=value", { headers: {} });
const cache = new Map();

// Handle a sample request.
handleRequest(request, cache);

// Output:
{
  "cacheKey": "https://example.com/resource?key=value",
  "cachedEntry": {
    "exists": false,
    "lastChecked": 1635879200000
  },
  "backendResponse": null // Backend check was not performed.
}
```

**Data:**

```json
{
  "cacheKey": "https://example.com/resource?key=value",
  "cachedEntry": {
    "exists": false,
    "lastChecked": 1635879200000
  },
  "backendResponse": {
    "status": 404
  }
}
```

**Flow:**

1. Cache Miss?
2. Backend Response
3. Yes
4. No
5. Yes
6. No
7. Receive Request
8. Check Cache
9. Fetch Backend
10. Backend 404?
11. Store in Cache as NegativeResult
12. Store in Cache as PositiveResult
13. C
14. Rate Limit Exceeded?
15. Return 429 Too Many Requests
16. Return 404 from Backend

