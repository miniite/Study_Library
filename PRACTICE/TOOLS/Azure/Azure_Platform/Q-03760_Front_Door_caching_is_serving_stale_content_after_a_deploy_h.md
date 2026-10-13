---
type: "practice"
difficulty: "Deep"
---

# Front Door caching is serving stale content after a deploy; how do purge, cache-control headers, and rules-engine caching interact, and how do you invalidate correctly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Azure DevOps and Artifact Governance](<../../../../TOOLS/Azure/Azure_Platform/Azure_DevOps_and_Artifact_Governance/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with Azure Front Door's caching mechanism, which can sometimes lead to serving stale content after a deploy due to misconfigurations in cache control headers or rules-engine settings. The goal is to understand how these components interact and ensure that we invalidate the correct content effectively.

#### Triage

The question revolves around three key areas:

Cache-Control Headers: These instruct caching mechanisms on whether and for how long to retain a resource.
Purging Cache: This involves explicitly invalidating cache entries.
Rules-Engine Caching: This allows customizing the caching behavior based on specific conditions.

#### Assumptions
We are working with Azure Front Door version 2 (Front Door).
The application has dynamic content that needs to be updated frequently, and we need to ensure that the latest content is served promptly.
We use cache control headers to manage caching behavior, and there's a need for fine-grained control over specific URLs or paths.

#### First-Principles Solution

Cache-Control Headers: These are HTTP headers used by both clients and servers to indicate whether content should be cached, and if so, for how long.

max-age indicates the maximum amount of time a resource can be cached.
no-cache tells the cache to revalidate the resource with the origin server before serving it.
no-store instructs caches not to store any version of the resource.

Purging Cache:

Azure Front Door supports purging individual URLs or entire paths using an API.
Purging can be done programmatically through REST APIs, PowerShell, or the Azure Portal.
BASH
az network front-door purge --front-door-name <front-door-name> --resource-group <resource-group>

Rules-Engine Caching:

Rules-engine caching allows us to define specific conditions under which certain URLs should be cached and for how long.
This is useful when you want more granular control over different parts of your website or API.
Step-by-Step Solution

Define Cache Control Headers:

HTTP
Cache-Control: max-age=3600, public
max-age sets the cache duration.
public indicates that this response is cacheable by all caches (not just private).

Set Up Purging in Azure Front Door:

Use REST API to purge specific URLs or paths after deployment:
BASH
az network front-door purge --front-door-name my-front-door --resource-group rg-dev --paths /api/v1/users /admin/*

Configure Rules-Engine Caching:

Define rules in the Azure Front Door's rules-engine to handle specific paths differently.
JSON
{
    "rules": [
        {
            "matchCondition": {
                "paths": ["/api/v1/products"],
                "methods": ["GET"]
            },
            "responseAction": "Cache",
            "cacheConfiguration": {
                "maxAgeInSeconds": 3600,
                "varyByHeaderNames": ["Accept-Encoding", "Cookie"]
            }
        },
        {
            "matchCondition": {
                "paths": ["/admin/*"],
                "methods": ["GET"]
            },
            "responseAction": "Cache",
            "cacheConfiguration": {
                "maxAgeInSeconds": 300,
                "varyByHeaderNames": []
            }
        }
    ]
}

Invalidate Correctly:

After deploying new content, use the purging mechanism to invalidate specific paths.
Ensure that rules-engine caching does not interfere with immediate updates by carefully defining vary-by headers.

#### Closing Summary

Proper configuration of cache control headers, purging mechanisms, and rules-engine caching is crucial for ensuring that Azure Front Door serves the latest content. By understanding how these components interact, you can effectively manage caching behavior to meet your application's needs.

#### Common Mistakes/Follow-Ups
Misconfigured Cache-Control Headers: Ensure they are set correctly to avoid stale content.
Incomplete Purging: Always purge specific paths or URLs after a deploy to ensure the latest content is served.
Rules-Engine Interference: Carefully define rules-engine caching to avoid conflicts with immediate updates.

**Code:**

```bash
# Example of purging cache using Azure CLI
az network front-door purge --front-door-name my-front-door --resource-group rg-dev --paths /api/v1/users /admin/*
```

**Flow:**

1. Define Cache Control Headers
2. Purge Specific Paths
3. Configure Rules-EngineCaching
4. Publish New Content
5. Publish New Content
6. Manually Purge Cache ifNeeded

