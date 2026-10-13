---
type: "practice"
difficulty: "Intermediate"
---

# Your SPA aggressively caches GET responses, and after a deploy that changed the response shape users see broken UI until a hard refresh. How do you version cached responses and bust the client cache safely?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to ensure that our SPA (Single Page Application) can handle changes in API response shape without breaking the UI due to aggressive caching. The key is to implement a versioning strategy for REST API responses and safely bust client-side cache.

#### Triage

The question is asking how to manage cached responses effectively when there's a change in the API response structure, ensuring that users see the updated content without requiring a hard refresh every time. This involves understanding caching mechanisms, HTTP headers, and strategies like using versioning parameters or ETag/Last-Modified headers.

#### First Principles Solution
Use Versioning Parameters: Add a version parameter to your API endpoints.
Cache-Control Headers: Configure appropriate Cache-Control headers on the server side.
Bypass Cache Mechanisms: Implement mechanisms to bypass or invalidate cache when necessary.
Step-by-Step Solution:
Add Versioning Parameter:
Modify API endpoints to include a version parameter in URLs, e.g., /api/v1/data.
This ensures that different versions of the response are cached separately and won't conflict if the structure changes.
Configure Cache-Control Headers:
Set Cache-Control headers appropriately on your server responses. For example, you can set a cache duration for non-versioned requests and handle versioned requests differently.
Bypass Cache Mechanisms:
Use conditional GETs with ETag or Last-Modified headers to check if the cached content is still valid.
Implement a mechanism to invalidate or bypass cache during deployments.

#### Example
Implementation:

API Endpoint with Versioning:

HTTP
GET /api/v1/data HTTP/1.1
Host: example.com

Server-Side Cache-Control Headers:

PLAINTEXT
Cache-Control: max-age=3600, s-maxage=86400, public, stale-while-revalidate=900, stale-if-error=3600
max-age sets the maximum time a resource can be cached.
s-maxage applies to shared (proxies) caches and is used for aggressive caching by CDNs.
public indicates that the response can be stored in any cache.
stale-while-revalidate allows clients to use stale content while revalidating with the server.
stale-if-error ensures that stale content will be used even if the revalidation fails.

Client-Side Cache Handling:

In your SPA, add logic to detect version changes and update the URL accordingly.
Use conditional requests (e.g., ETag) to check cache validity before making a new request.

#### Example
Code Snippet:
JAVASCRIPT
// Fetch data with versioning
async function fetchData() {

#### cons
t url = `/api/v1/data?v=${new Date().getTime()}`;

#### cons
t response = await fetch(url);

  if (!response.ok) {
    throw new Error('Network response was not ok');
  }

  return response.json();
}

// Use conditional requests to check cache validity
async function safeFetchData() {

#### cons
t url = `/api/v1/data`;

#### cons
t headers = { 'If-None-Match': localStorage.getItem('etag') };

  try {

#### cons
t response = await fetch(url, { headers });

    if (response.status === 304) {
      // Cache is still valid
      return JSON.parse(localStorage.getItem('cachedData'));
    } else {
      // Cache has been invalidated or content has changed

#### cons
t data = await response.json();

      localStorage.setItem('etag', response.headers.get('ETag'));
      localStorage.setItem('cachedData', JSON.stringify(data));

      return data;
    }
  } catch (error) {

#### cons
ole.error('Error fetching data:', error);
  }
}

#### Closing Summary

By implementing versioning parameters, configuring appropriate Cache-Control headers, and using conditional requests with ETag or Last-Modified headers, we can safely manage cached responses and avoid breaking the UI during API changes. This approach ensures that users see updated content without requiring a hard refresh.

#### Common Mistakes/Follow-Ups
Not Handling Versioning Properly: Failing to add version parameters to URLs can lead to caching conflicts.
Inadequate Cache-Control Headers: Incorrect or overly restrictive cache-control settings can hinder performance.
Lack of Conditional Requests: Not using ETag/Last-Modified headers can result in unnecessary revalidations and increased load on the server.

Always test thoroughly, especially during deployments, to ensure that the cache behavior is as expected.

**Code:**

```javascript
// Example code for fetching data with versioning
async function fetchData() {
  const url = `/api/v1/data?v=${new Date().getTime()}`;
  const response = await fetch(url);

  if (!response.ok) {
    throw new Error('Network response was not ok');
  }

  return response.json();
}

// Example code for safe fetching with conditional requests
async function safeFetchData() {
  const url = `/api/v1/data`;
  const headers = { 'If-None-Match': localStorage.getItem('etag') };

  try {
    const response = await fetch(url, { headers });

    if (response.status === 304) {
      // Cache is still valid
      return JSON.parse(localStorage.getItem('cachedData'));
    } else {
      // Cache has been invalidated or content has changed
      const data = await response.json();

      localStorage.setItem('etag', response.headers.get('ETag'));
      localStorage.setItem('cachedData', JSON.stringify(data));

      return data;
    }
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}
```

**Data:**

```plaintext
# Sample API Response Data
{
  "id": 1,
  "name": "Example Item",
  "description": "This is an example item."
}

# Sample ETag Header
ETag: "abc123"
```

**Flow:**

1. Yes
2. No
3. Initiate Request
4. Is Cache Valid?
5. Return Cached Data
6. Fetch Fresh Data
7. Store New ETag and Data inCache
8. Return Fresh Data

