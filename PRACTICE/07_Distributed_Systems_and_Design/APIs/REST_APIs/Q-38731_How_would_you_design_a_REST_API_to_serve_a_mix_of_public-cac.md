---
type: "practice"
difficulty: "Deep"
---

# How would you design a REST API to serve a mix of public-cacheable and private-per-user data on the same logical page, choosing what to split into separately-cacheable endpoints for CDN efficiency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HTTP Caching and ETags](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/HTTP_Caching_and_ETags.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a REST API that efficiently serves both public-cacheable and private-per-user data from the same logical page, we need to ensure high performance while maintaining security and cache coherence. The key is to split the data into separately-cacheable endpoints where appropriate, leveraging HTTP caching mechanisms like ETags and Cache-Control headers.

#### Triaging the Question

The interviewer is probing your understanding of REST principles, HTTP caching, and how to balance user privacy with performance optimization. You'll need to consider:

How to distinguish between public and private data.
Which parts can be cached, and which cannot.
Ensuring security while enabling efficient caching.
Designing the API

First-principles thinking starts by understanding that a single page often contains both static content (like images or HTML) and dynamic, user-specific content. To optimize for CDN efficiency, we need to separate these concerns:

Identify Public vs Private Data:

Static assets (images, CSS, JS files) can be publicly cached.
User-specific data (user profiles, session information) must be served privately.

Use HTTP Headers for Caching:

Set Cache-Control: public and ETag headers on cacheable resources to enable CDN caching.
Use Cache-Control: private, no-cache or Authorization header checks to ensure sensitive data is not cached publicly.

Endpoint Design:

Create separate endpoints for different types of data to control caching behavior precisely.
MARKDOWN
GET /public-assets -> Serves public cacheable assets
GET /private-data/{userId} -> Serves user-specific private data
For a mix of both, consider using sub-resources or combining them in a way that respects caching rules.

Session Handling and Authentication:

Use tokens (JWT) for authentication to avoid exposing session information.
Securely handle sensitive operations like modifying user profiles by requiring proper authorization checks.

#### Example
Workflow

Let's walk through an example where we have a page with both public assets and private data:

Fetching the Main Page HTML:

HTTP
GET /page/{userId}
This request would include all necessary HTML content.
Public assets like images and CSS files are served via separate endpoints.

Serving Public Assets:

HTTP
GET /public-assets/image.png
HTTP headers: Cache-Control: public, max-age=31536000 (cache for one year).

Fetching User-Specific Data:

HTTP
GET /private-data/{userId}
This request would return user-specific data like profile settings.
HTTP headers might include Cache-Control: private, no-cache.

#### Closing Summary
and Common Mistakes

By carefully structuring your API endpoints and using appropriate caching mechanisms, you can achieve a balance between performance and security. A common mistake is to neglect proper session handling or misconfigure cache directives, leading to either poor performance or exposure of sensitive data.

Mistake 1: Forgetting to use Cache-Control headers properly.
Mistake 2: Not implementing robust authentication mechanisms for private endpoints.
Mistake 3: Over-caching user-specific data, potentially exposing it on the CDN.

**Comparison:**

| Resource Path | Cache Control | Description |
|---------------|---------------|-------------|
| /public-assets/image.png | public, max-age=31536000 | Public asset that can be cached by CDN |
| /private-data/{userId} | private, no-cache | User-specific data that must not be cached |

**Code:**

```
HTTP
Copy
GET /public-assets/image.png
HTTP/1.1 200 OK
Content-Type: image/png
Cache-Control: public, max-age=31536000

<Binary Image Data>

GET /private-data/{userId}
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: private, no-cache

{"username": "john_doe", "email": "johndoe@example.com"}
```

**Example input / output:**

```text
Input:
GET /page/123

Output:
200 OK - HTML content with links to public assets and a request for private data.

Intermediate Requests:
GET /public-assets/image.png -> 200 OK, Cache-Control: public, max-age=31536000
GET /private-data/123 -> 200 OK, Cache-Control: private, no-cache

Output:
HTML content with cached image and fetched user data.
```

**Data:**

```json
{
  "publicAssetsPath": "/public-assets",
  "privateDataPath": "/private-data/{userId}",
  "cacheControlHeaders": {
    "publicAsset": "public, max-age=31536000",
    "privateData": "private, no-cache"
  }
}
```

**Flow:**

1. Yes
2. No
3. User Request
4. Is Request for Public Asset?
5. Cached from CDN
6. Check Authentication
7. Retrieve Private Data
8. Return Response withCache-Control: private,no-cache
9. Send HTML Main Page
10. Include Links to Public Assetsand Private Data Endpoints
11. Cache Response from CDN
12. Cache Not Enabled forUser-Specific Data

