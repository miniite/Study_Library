---
type: "practice"
difficulty: "Deep"
---

# You have a React SPA served from a CDN and a separate API. After a deploy some users get a white screen because their cached index.html points at a JS chunk that no longer exists. How do you design the deploy and cache strategy to prevent this?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CDN Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/CDN_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of users experiencing a white screen due to outdated JavaScript chunks, we need to design an effective deployment and caching strategy that ensures consistency between our static assets (like the index.html file) and the dynamic server-side components. The goal is to minimize disruptions while maintaining high performance.

The key challenge here lies in managing the lifecycle of cached content across both the CDN and the origin servers. We'll break down the solution from first principles, considering the trade-offs involved in caching strategies, and then propose a robust design that balances these factors.

#### Triage

This question is probing your understanding of how to manage static and dynamic assets efficiently in a modern web application setup. The interviewer wants to see how you balance the need for rapid deployment with the necessity of preventing stale content issues. Key assumptions include:

We are using a CDN (Content Delivery Network) to serve our React SPA.
The API is separate from the frontend, meaning we have distinct responsibilities and caching rules.
Users experience a white screen when their cached index.html points to non-existent JavaScript chunks.

#### Solution
Versioned URLs: Use versioned URLs for your static assets (e.g., index.html?v=20231018). This ensures that each deployment results in new, unique URLs.
Cache Headers: Configure appropriate cache headers on both the origin server and CDN to control how long cached content is retained. For example:
Set a short max-age for dynamic content (e.g., 5 minutes).
Use ETags or Last-Modified headers with longer TTLs for static assets.
Edge Caching: Leverage edge caching on the CDN to reduce load times and server stress but ensure it doesn't serve stale data. Configure cache invalidation policies to invalidate the cache when updates are made.
Brotli Compression: Compress your assets using Brotli or Gzip to save bandwidth and improve performance.
Edge Side Includes (ESI): If you're using a CDN that supports ESI, use it to dynamically include parts of the page without invalidating the whole page cache.

#### First-Principles Solution
Versioned URLs

By using versioned URLs for both index.html and its JavaScript chunks, we ensure that when we deploy new code, users will always get a fresh copy. For example:

PLAINTEXT
https://cdn.example.com/app.js?v=20231018

When deploying, update the version number to force browsers to fetch new versions.

Cache Headers

Configure cache headers appropriately on both origin and CDN:

Origin Server: Set appropriate HTTP headers for static content.
HTTP
# For index.html (versioned)
Cache-Control: max-age=3600, public

# For JavaScript chunks
Cache-Control: max-age=86400, public; immutable
CDN: Ensure CDN respects these headers and provides additional layer of caching.
HTTP
X-Accel-Expires: 1h (for dynamic content)
Cache-Control: public, max-age=3600 (for static content)
Edge Caching and Invalidations

Implement cache invalidation policies on the CDN:

Use a strategy like Cache-Control with max-age to limit how long cached content is retained.
Implement dynamic cache invalidation for critical changes, such as when new JavaScript chunks are deployed.

#### Closing Summary

By using versioned URLs, appropriate cache headers, and leveraging edge caching with proper invalidation strategies, we can ensure that users consistently receive the latest and most relevant content. This approach balances performance optimizations with robust handling of content updates, thereby minimizing disruptions to end-users.

#### Common Mistakes/Follow-Ups
Not Using Versioned URLs: Failing to version your assets leads to stale content issues.
Inadequate Cache Headers: Setting overly long cache times for static assets can result in users seeing outdated content longer than necessary.
Lack of Dynamic Invalidation Policies: Not invalidating the CDN cache when critical changes are made can lead to serving old content.

For follow-up questions, consider discussing how to handle edge cases like partial deployments or ensuring consistent behavior across different CDNs. Discussing additional best practices and technologies (like Cloudflare Workers) could further enhance the solution.

**Comparison:**

|Asset| Cache Control|
|---|---|
|index.html| max-age=3600, public|
|app.js (chunk)| X-Accel-Expires: 1h, Cache-Control: public, max-age=3600|

**Code:**

```
HTTP
Copy
# Example of appropriate cache headers for static assets
Cache-Control: max-age=3600, public

# Example of cache headers with dynamic content
X-Accel-Expires: 1h
Cache-Control: public, max-age=3600
```

**Example input / output:**

```text
# Example request and response headers for cache control

Request:
GET /app.js HTTP/1.1
Host: cdn.example.com
Cache-Control: max-age=0, no-cache
X-Cache-Key: v20231018

Response:
HTTP/1.1 200 OK
Content-Type: application/javascript; charset=utf-8
Cache-Control: public, max-age=3600
Last-Modified: Fri, 19 Oct 2023 00:00:00 GMT
ETag: "v20231018"
```

**Data:**

```json
{
  "cdnConfig": {
    "cacheControl": {
      "staticAssets": "max-age=3600, public",
      "dynamicContent": "X-Accel-Expires: 1h, Cache-Control: public, max-age=3600"
    }
  },
  "deploymentStrategy": {
    "versioning": "https://cdn.example.com/app.js?v={version}",
    "cacheInvalidation": "Use X-Cache-Key header for ESI invalidations"
  }
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TB
3. A[User Request] --> B(CDN Cache)
4. B --> C-Origin Server
5. C --> D-Versioned URL Check
6. D -- Yes --> E[Serve Latest Content]
7. D -- No --> F-Cache Invalidation
8. F --> G-CDN Invalidation
9. G --> H[Serve Updated Content]

