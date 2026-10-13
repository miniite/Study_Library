---
type: "practice"
difficulty: "Intermediate"
---

# You cache API responses behind an ETag, but your load balancer terminates TLS and rewrites headers. How do you make sure the ETag stays stable and revalidation actually saves bandwidth end to end?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the ETag remains stable and revalidation saves bandwidth end-to-end, we need to address a few key aspects: understanding how TLS termination affects ETags, ensuring consistent header rewriting by the load balancer, and implementing efficient caching strategies on both the client and server sides. Let's break this down from first principles.

#### Triaging the Question

The question is primarily about maintaining the stability of the ETag in an environment where the load balancer terminates TLS and rewrites headers. The core goal is to ensure that revalidation requests are accurate, and thus save bandwidth by serving cached content when possible. This involves:

Understanding how header rewriting works.
Ensuring consistent and correct ETags across different client requests.
Configuring caching mechanisms effectively.

#### First-Principles Solution
Understand Header Rewriting: The load balancer rewrites headers, which can affect the ETag if not handled correctly. We need to ensure that the ETag is rewritten consistently and accurately so it remains valid for revalidation requests.
ETag Stability: The ETag should remain stable even after TLS termination and header rewriting. This means the value of the ETag must be derived from a source that is consistent across all client requests and not altered by the load balancer or any other layer in the system.
Caching Strategy:
Implement a caching strategy on both the client (browser) side using HTTP caching headers like Cache-Control and ETag.
Ensure server-side caching respects the ETag for revalidation requests.
Step-by-Step Solution

Configure Load Balancer to Preserve ETags:

Configure the load balancer to pass through the original ETag from the backend service without rewriting it.
Use a consistent hashing mechanism to ensure that requests with the same URL and query parameters always hit the same backend instance, maintaining consistency in generating ETags.

Generate Stable ETags on the Server:

Ensure that the server-side logic for generating ETags is stateless and based solely on request content.
Use a combination of request headers (e.g., If-None-Match), URL path, query parameters, and possibly cookies or tokens to generate unique but stable ETags.

Client-Side Caching:

Configure the browser to cache responses using HTTP caching directives like Cache-Control: max-age and ETag.
Ensure that the client sends a revalidation request with the correct If-None-Match header when accessing cached content.

Server-Side Validation:

Implement a validation mechanism on the server to check if the ETag in the If-None-Match header matches the current version of the resource.
If the ETags match, return a 304 Not Modified response instead of sending the full content.

#### Closing Summary
and Common Mistakes

In summary, ensuring stable ETags and effective revalidation involves configuring the load balancer to pass through original headers correctly, generating consistent ETags on the server side, and configuring both client and server-side caching strategies. Common mistakes include:

Failing to preserve ETags across TLS termination points.
Incorrectly rewriting or altering ETags by the load balancer.
Misconfiguring cache invalidation rules leading to stale content being served.
Closing Note

This approach ensures that revalidation requests are accurate, and cached content is served when possible, thereby saving bandwidth end-to-end. Careful configuration and testing are essential to avoid common pitfalls.

**Comparison:**

```
Scenario	Expected Outcome
Initial Request	Server generates ETag and responds with content.
Subsequent Request (with If-None-Match)	If ETags match, server returns 304 Not Modified; otherwise, it serves fresh content.
```

**Flow:**

1. Client Requests Resource
2. Server Generates ETag
3. ETag Returned with Response
4. Client Stores ETag Locally
5. Client Makes SubsequentRequest
6. Client Adds If-None-MatchHeader
7. Server Checks ETag
8. 304 Not Modified if ETagsMatch
9. Fresh Content Served if ETagsDon't Match

