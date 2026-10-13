---
type: "practice"
difficulty: "Deep"
---

# You cut over a high-traffic API to a new load balancer via DNS but 15% of clients stayed on the old endpoint for hours despite a 60s TTL. How do you explain and prevent this?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ DNS TTLs](<../../../../06_Cloud_Computing/Cloud_Foundations/Networking/DNS_TTLs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a DevOps context, when you switch an API to a new load balancer using DNS with a short Time-to-Live (TTL) like 60 seconds, it's expected that most clients would quickly update their cache and route traffic to the new endpoint. However, if only 15% of users remain on the old endpoint despite the short TTL, this can be explained by caching mechanisms at different layers: DNS servers, client-side caches, CDN (if used), and even network infrastructure. To prevent such a situation in the future, we need to understand why these clients are not updating their records and then implement strategies to ensure consistent updates.

Let's break down the issue step-by-step:

Understanding Client Caching: Clients can cache DNS resolution results for extended periods beyond the TTL due to various reasons. For example, many browsers cache DNS resolutions aggressively, often keeping them cached for several minutes or even hours.
DNS Server Caches: DNS resolvers and recursive name servers might also cache records based on their configuration and policies.
CDN Involvement (if applicable): If a Content Delivery Network (CDN) is in use, it could have its own caching mechanism that might not respect the TTL from your DNS provider.

#### Triage

The interviewer is probing into understanding the intricacies of DNS caching mechanisms and their impact on load balancing. They are also interested in strategies to mitigate caching issues and ensure a smooth transition during such events.

#### First Principles Solution
Client-Side Caching: The majority of client-side DNS caches (browsers, mobile apps) may not update for up to an hour or even longer due to aggressive caching policies.
DNS Resolver Caches: Recursive name servers and public DNS resolvers like Google Public DNS might cache the old IP for a while, especially if they frequently resolve queries from these IP addresses.
CDN Cache: If you are using a CDN, it will have its own cache that could hold onto the old IP even after your TTL expires.
Prevention Strategies
Increase DNS TTL Temporarily: Before the cut-over, temporarily increase the DNS TTL to several hours or days to allow enough time for aggressive client-side and public DNS caches to be flushed.
Flush Caches: Use tools like dig, nslookup, and host commands from multiple locations (public DNS resolvers) to flush DNS caches before the cut-over.
CDN Cache Management: If using a CDN, ensure that its cache is invalidated or refreshed after the TTL expires on your main DNS records.
Monitoring Tools: Implement monitoring tools like Cloudflare’s built-in DNS health checks and visibility into cached responses from different resolver pools.

#### Closing Summary

In conclusion, to prevent clients from staying on an old endpoint post-cutover, it's crucial to understand that caching is widespread at various levels and requires proactive measures such as temporary increases in TTL, cache flushing, and CDN management. A combination of these strategies ensures a smooth transition without significant disruptions.

#### Common Mistakes/Follow-Ups

Not Considering Client Caches: Clients might have aggressive DNS cache policies.
Underestimating Resolver Cache Lifetimes: Public DNS resolvers may take longer to update their records.
Ignoring CDN Management: If using a CDN, not invalidating its cache can lead to persistent issues.

**Comparison:**

```
Resolver	Flush Status	Timestamp
Google Public DNS	Flushed	2023-10-15T14:30Z
Cloudflare DNS	Flushed	2023-10-15T14:35Z
```

**Code:**

```python
# Example code snippet for flushing DNS cache on different resolvers (Linux)
import subprocess

def flush_dns(resolver):
    command = f"echo \"| ipconfig /flushdns\" | {resolver}"
    output = subprocess.check_output(command, shell=True)
    print(output.decode())

# Flush caches from multiple public DNS resolvers
flush_dns("nslookup -type=A example.com 8.8.8.8")
flush_dns("nslookup -type=A example.com 1.1.1.1")
```

**Example input / output:**

```text
Request to flush DNS cache on Google Public DNS:
> nslookup -type=A example.com 8.8.8.8

Response:
Non-authoritative answer:
Name:    example.com
Address:  93.184.216.34

Request to flush DNS cache on Cloudflare DNS:
> nslookup -type=A example.com 1.1.1.1

Response:
Non-authoritative answer:
Name:    example.com
Address:  93.184.216.34
```

**Data:**

```json
{
  "dns_flush_results": [
    {
      "resolver": "Google Public DNS (8.8.8.8)",
      "status": "Cache Flushed",
      "timestamp": "2023-10-15T14:30:00Z"
    },
    {
      "resolver": "Cloudflare DNS (1.1.1.1)",
      "status": "Cache Flushed",
      "timestamp": "2023-10-15T14:35:00Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Initial TTL
5. Is Client Cache Aggressive?
6. Increase TTL Temporarily
7. Flush DNS Caches on MultipleResolvers
8. Is CDN Used?
9. Invalidate CDN Cache
10. Maintain Monitoring Tools
11. Monitor Transition
12. Smooth Transition Confirmed

