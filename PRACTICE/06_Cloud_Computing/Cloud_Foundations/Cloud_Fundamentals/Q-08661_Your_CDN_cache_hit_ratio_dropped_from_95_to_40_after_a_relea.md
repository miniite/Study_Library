---
type: "practice"
difficulty: "Deep"
---

# Your CDN cache hit ratio dropped from 95% to 40% after a release. How do you find what changed and restore cacheability?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CDN Strategy](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/CDN_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the drop in cache hit ratio from 95% to 40%, we need to methodically diagnose the issue, identify changes introduced by recent releases, and then evaluate potential fixes to restore cacheability. This involves a deep dive into both application behavior and cloud infrastructure.

The core question here is: How can we efficiently pinpoint changes that impacted our CDN's caching mechanism? The interviewer is probing your ability to analyze system logs, understand HTTP responses, and navigate the complex interactions between application code, network configuration, and cloud services.

Let’s break it down step-by-step:

#### Assumptions
and Context:

We have a web application using a CDN for content delivery.
The drop in cache hit ratio is significant (from 95% to 40%), indicating that many requests are now being served from the origin server instead of the cached content on the CDN.

Initial Steps:

Review Recent Changes: Examine any code changes, configuration updates, or deployments made recently.
Check Logs and Metrics: Analyze access logs, CDN logs, and application metrics to understand recent behavior patterns.
CDN Configuration Review: Ensure that the CDN cache settings (TTLs, cache keys, etc.) haven’t been inadvertently altered.

Systematic Approach:

Identify Cache Invalidation Patterns: Look for patterns in requests that are causing frequent cache invalidation.
Analyze HTTP Headers and Responses: Inspect the HTTP headers (like Cache-Control, ETag) to ensure they are correctly set.
Network Performance Monitoring: Use tools like Cloudflare’s Speed Insights or AWS CloudFront logs to understand network latency and request behavior.

Troubleshooting:

Check for Content Updates: Ensure that static content updates have been properly deployed without inadvertently invalidating the cache.
Evaluate API Endpoints: If your application uses dynamic content, review how API endpoints are being handled by the CDN.
Assess Application Code Changes: Check if recent code changes in backend services or front-end frameworks might be affecting caching behavior.

Implement Fixes and Monitor:

Once potential issues are identified, implement necessary fixes (e.g., reconfigure cache settings, update HTTP headers).
Continuously monitor the CDN’s performance metrics to ensure improvements.
Set up alerts for unusual traffic patterns or high request volumes that could indicate future issues.

#### Closing Summary
and Common Mistakes:

By methodically reviewing recent changes and system configurations, we can identify and correct issues impacting cacheability.
Common pitfalls include overlooking subtle configuration changes, misinterpreting log data, or failing to consider the interplay between different layers of the application stack.

**Comparison:**

```
Timestamp	Event	Request URL	Response Code
2023-10-01T14:30:00Z	cache miss	/api/v1/users	200
2023-10-01T15:00:00Z	cache hit	/static/css/main.css	200
```

**Code:**

```python
# Example code snippet for logging and metrics analysis
import requests

def analyze_logs():
    response = requests.get('https://example.com/log/api')
    logs = response.json()
    # Analyze logs for patterns in cache misses
```

**Example input / output:**

```text
Input:
Access logs for a period post-release, with headers and statuses.

Output:
Detailed analysis of cache misses and hits, patterns in request URLs, and specific HTTP header fields (e.g., Cache-Control) that indicate potential issues.
```

**Data:**

```json
{
  "log_sample": [
    {"timestamp": "2023-10-01T14:30:00Z", "event": "cache miss", "request_url": "/api/v1/users"},
    {"timestamp": "2023-10-01T15:00:00Z", "event": "cache hit", "request_url": "/static/css/main.css"}
  ]
}
```

**Flow:**

1. Identify Recent Changes
2. Review Logs and Metrics
3. Evaluate CDN Configuration
4. Check for Content Updates
5. Analyze HTTP Headers andResponses
6. Evaluate API Endpoints
7. Assess Application CodeChanges
8. Implement Fixes and Monitor
9. Continuous Monitoring

