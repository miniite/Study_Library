---
type: "practice"
difficulty: "Deep"
---

# How do CDN cache-purge and CDN cache-tag invalidation work when you push new content?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Caching is a crucial optimization technique that reduces latency by storing frequently accessed data closer to the user. Content Delivery Networks (CDNs) employ caching to speed up content delivery across geographical locations. However, managing caches when new content is pushed can be complex. This answer will explain how CDNs handle cache-purge and cache-tag invalidation, focusing on both practical implementation and strategic considerations.

CDN caching typically involves storing copies of frequently accessed data at multiple points around the world to minimize latency. When a user requests an asset (e.g., a webpage, image), the CDN serves it from its closest edge node rather than directly from the origin server.

Cache-purge operations are used to invalidate specific pieces of cached content when changes occur. Cache-tag invalidation is another mechanism that helps manage cache coherence by tagging resources and signaling them for removal or update upon change.

To understand how these mechanisms work, we need to consider the lifecycle of a resource in a CDN and the challenges in maintaining cache consistency across multiple edge nodes.

**Comparison:**

```
Resource	URL	Etag
Image	/images/new-logo.png	v1235
Cache Tag	Identifier	Timestamp
Logo	logo-update-v2	2023-10-05T14:30:00Z
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Resource is requested] --> B[Check Cache];
    B --> C(Cache Hit);
    C --> D[Serve Cached Content];
    B --> E(Cache Miss) --> F[Send Request to Origin Server];
    F --> G[Process and Send New/Updated Content];
    G --> H[Update CDN Edge Nodes];
    H --> I[Publish Cache Tag];
```

**Example input / output:**

```

When a new image is uploaded to the origin server with an updated etag, the CDN performs the following steps:

Cache-Purge: The origin server sends a cache purge request to the CDN for the specific URL.
Tag Update: A new cache tag (e.g., logo-update-v2) is assigned and published to all edge nodes.
Edge Node Update: Upon receiving the update, each edge node checks if any cached content has the same identifier as the new tag.
Purge or Invalidate: If a match is found, the cache entry for that resource is invalidated and removed from the edge node.

Sample input:

Origin Server Request:
HTTP
Copy
PUT /images/new-logo.png HTTP/1.1
Host: origin.example.com
Etag: v1235

<New Image Data>

Sample output:

CDN Response:
JSON
Copy
{
  "status": "OK",
  "message": "Cache tag 'logo-update-v2' updated."
}
```

**Data:**

```json
{
  "resource": {
    "url": "/images/new-logo.png",
    "etag": "v1234"
  },
  "cacheTag": {
    "identifier": "logo-update-v2",
    "timestamp": "2023-10-05T14:30:00Z"
  }
}
```

**Flow:**

1. Resource Request
2. Check Cache
3. Cache Hit
4. Serve Cached Content
5. Cache Miss
6. Send Request to Origin Server
7. Process and SendNew/Updated Content
8. Update CDN Edge Nodes
9. Publish Cache Tag

