---
type: "practice"
difficulty: "Intermediate"
---

# Your cache and DB disagree after a deploy. Describe your step-by-step approach to diagnose the inconsistency.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Cache Invalidation](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a scenario where cached data disagrees with database (DB) records, it's crucial to methodically investigate the issue to determine the root cause and potential solutions. As an intermediate experience Backend SDE-2/3 or Architect/Principal, your approach should be systematic and consider various factors like cache invalidation policies, transactional consistency, and recent changes in the application.

Firstly, let's summarize what we're dealing with: a scenario where the cache contains outdated information while the database reflects the latest state. This discrepancy could arise due to improper cache management or issues during deployment.

Triaging the Problem
Identify the Scope: Determine which specific data is affected and whether it’s a single piece of data or multiple entries.
Check Recent Changes: Review recent deployments, changes in code, and any new features that might interact with the cache or database.
Verify Configuration Settings: Confirm that your caching mechanisms are correctly configured to invalidate and refresh caches when needed.
Step-by-Step Solution

Review Cache Invalidation Policies:

Cache Types and Stores: Determine which types of data are cached (e.g., in-memory, Redis) and how they are stored.
Invalidate Rules: Check if there are rules or triggers for invalidating cache entries after a deployment. Common methods include TTL-based expiration, explicit invalidation via APIs, or trigger events from the database.

Database Changes:

Transaction Logs: Review recent transactions in the database to see what changes were made.
Deployment Records: Look at logs and documentation of the latest deployment to ensure that all necessary steps were followed, including any manual cache invalidations.

Code Inspection:

Cache Operations: Inspect the relevant parts of the codebase where caching operations occur. Ensure that calls to invalidate caches are being made correctly.
Concurrency Issues: Consider if there might be concurrent access or race conditions between database updates and cache reads or writes.

Logs and Monitoring Tools:

Application Logs: Check logs for any errors, warnings, or messages related to caching during the deployment period.
Performance Metrics: Use monitoring tools to track performance metrics like cache hit rates and latency before and after the deploy.

Manual Testing:

Simulate Scenario: Manually trigger a scenario that leads to data updates in both the database and cached data to observe behavior.
Edge Cases: Test edge cases such as partial updates or high concurrency scenarios to see if they affect cache consistency.

#### Closing Summary

By methodically following these steps, you can identify the root cause of the inconsistency between your cache and DB. Common mistakes include improper invalidation policies, overlooked changes in deployment processes, and failing to test for edge cases adequately.

#### Common Mistakes/Follow-ups
Failing to Log Properly: Not logging critical operations like cache invalidations or database queries can make troubleshooting difficult.
Assuming Invalidation Policies Worked: Without concrete evidence from logs or tests, assuming that caching mechanisms worked correctly can lead to overlooking actual issues.
Ignoring Deployment Scripts: Manual steps or scripts used during deployment might not be properly documented or executed.

**Comparison:**

```
Operation	Expected Behavior	Actual Behavior
Cache Invalidation	Triggers	Does not trigger
Database Update	Successful	Success
```

**Code:**

```python
# Example of a cache invalidation function in Python using Redis
import redis

def invalidate_cache(key):
    r = redis.Redis(host='localhost', port=6379, db=0)
    r.delete(key)

# Example usage during deployment
invalidate_cache('user_data_123')
```

**Example input / output:**

```text
Input:
- Deploy new feature with database changes.
- Cache invalidation not properly triggered for affected keys.

Output:
- Cache remains stale, showing outdated information.
- Database reflects updated state after deployment.
```

**Data:**

```json
{
  "cache_key": "user_data_123",
  "db_table": "users",
  "last_updated_at": "2023-10-05T14:00:00Z"
}
```

**Flow:**

1. No
2. Yes
3. Database Update
4. Cache Invalidate?
5. Cache Consistent
6. Cache Invalidation Triggers
7. Cache Refreshes
8. Cache Consistent

