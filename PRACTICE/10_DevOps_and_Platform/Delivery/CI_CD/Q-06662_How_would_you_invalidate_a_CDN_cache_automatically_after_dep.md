---
type: "practice"
difficulty: "Deep"
---

# How would you invalidate a CDN cache automatically after deploying new frontend assets?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI Caching](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To invalidate a CDN cache automatically after deploying new frontend assets, we need to ensure that users and downstream systems receive the latest versions of our static content without unnecessary latency. This process involves understanding how CDNs operate, leveraging web standards like HTTP headers, and integrating with CI/CD pipelines.

The core idea is to send specific signals from your deployment system (CI/CD pipeline) to the CDN, instructing it to clear or refresh its cache for certain URLs or file types. Here’s a step-by-step approach:

Understand CDNs and Their Caching Mechanisms: Modern CDNs use complex caching strategies based on HTTP headers, URL patterns, and content hashes.
Identify the Signals Needed: We need to emit HTTP ETags or If-None-Match headers with our new assets so that clients can check if a local cache is still valid. Additionally, we may send explicit cache invalidation requests via custom HTTP headers like Cache-Control and Expires.
Integrate with CI/CD Pipeline: Update the deployment scripts to include steps for cache invalidation as part of the post-deployment process.

Let’s break this down further:

#### Triage

The question is essentially asking about how to manage the caching lifecycle in a web application, specifically focusing on ensuring that new frontend assets are served without being intercepted by stale CDN caches. The interviewer might be probing your understanding of HTTP headers, cache control mechanisms, and automation practices.

#### Assumptions
You have access to an existing CI/CD pipeline.
You can modify deployment scripts or use additional tools/services to manage cache invalidation.
Your application serves static assets via a CDN (e.g., Cloudflare, Akamai).

#### Solution
from First Principles

Identify URLs and Assets: Determine which frontend assets are subject to frequent updates, such as CSS, JS files, and images.

Configure HTTP Headers:

Use ETag or Last-Modified headers for automatic cache invalidation.
Set appropriate Cache-Control directives (e.g., max-age, s-maxage) to control how long content can be cached locally.

Generate Versioned URLs: To ensure that clients always fetch the latest assets, generate versioned URLs in your deployment scripts. For example:

HTML
<script src="/static/app.js?v=123456789"></script>

Cache Invalidation via Custom Headers:

After deploying new assets, send a POST request to the CDN with custom headers like X-Cache-Control: invalidate.
This triggers the CDN to purge cached copies of specific URLs or files.

Automate Cache Invalidations:

Integrate cache invalidation steps into your CI/CD pipeline using tools like AWS Lambda@Edge, Vercel PurgeCache API, or Cloudflare Workers.

#### Example
in a CI/CD script (e.g., GitHub Actions):
YAML
steps:
  - name: Deploy and Invalidate Cache
    run: |
      echo "Deploying new assets..."
      # Your deployment command here

      echo "Invalidating CDN cache..."
      curl -X POST https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/purge_cache \
        -H "Authorization: Bearer <API_TOKEN>" \
        -d '{"files": ["/static/app.js?v=123456789"]}'

#### Closing Summary

By integrating cache invalidation steps into your CI/CD pipeline, you can ensure that users always receive the latest frontend assets without hitting stale versions in the CDN. This approach requires understanding HTTP caching mechanisms and leveraging both static asset versioning and dynamic cache invalidation signals.

#### Common Mistakes/Follow-Ups
Mistake: Failing to use versioned URLs or ETags, leading to accidental reuse of outdated content.

#### Follow-Up
Discuss edge cases like handling large-scale deployments where CDN purges might take longer, requiring fallback strategies or prioritizing critical assets first.

**Code:**

```yaml
steps:
  - name: Deploy and Invalidate Cache
    run: |
      echo "Deploying new assets..."
      # Your deployment command here

      echo "Invalidating CDN cache..."
      curl -X POST https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/purge_cache \
        -H "Authorization: Bearer <API_TOKEN>" \
        -d '{"files": ["/static/app.js?v=123456789"]}'
```

**Example input / output:**

```text
Input:
- Deployed new version of app.js with a v=123456789 query parameter.
- Sent cache invalidation request to Cloudflare API.

Output:
- Successfully deployed new assets and invalidated CDN cache for /static/app.js?v=123456789.
```

**Flow:**

1. Deploy New Assets
2. Publish Versioned URLs
3. Generate Cache ControlHeaders
4. Send Cache InvalidationRequest to CDN
5. CDN Purges Stale Content
6. Serve Latest Version to Users

