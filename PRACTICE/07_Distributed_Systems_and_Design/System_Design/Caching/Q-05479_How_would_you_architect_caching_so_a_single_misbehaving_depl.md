---
type: "practice"
difficulty: "Deep"
---

# How would you architect caching so a single misbehaving deploy that pollutes the cache with bad data can be rolled back or purged quickly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Caching and Service Independence](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Caching_and_Service_Independence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Caching is crucial for performance optimization, but its management must account for potential issues like cache pollution from misbehaving deployments. To address this, we'll architect a robust caching strategy that includes versioning, invalidation mechanisms, and a mechanism to roll back or purge the cache quickly.

#### Triaging the Question

The question asks how to architect a caching system so that in case of a misbehaving deployment, the cache can be quickly rolled back or purged. The interviewer is probing your understanding of caching systems, versioning strategies, and fault-tolerance mechanisms.

#### First-Principles Solution

Versioned Caching:

Implement a version-based caching system where each deployment has its own unique identifier (e.g., v1, v2).
Use this version to prefix cache keys or store versions in the cache metadata.

Cache Invalidation Mechanism:

Design an invalidation policy that can invalidate all caches associated with a specific version.
For example, if a new version v3 is deployed, you could mark all older versions (v1, v2) as invalid and initiate their removal.

Cache Purge Mechanism:

Develop a mechanism to quickly purge the cache for a particular version or set of versions.
This can be done through an API that triggers the purging process, or via scheduled tasks.

Rollback Strategy:

Implement a rollback strategy where if a deployment fails and causes cache pollution, you can roll back to a previous healthy state by invalidating caches associated with the problematic version.
Ensure this rollback is seamless and does not impact ongoing operations.
Step-by-Step Solution:

Versioned Caching:

PYTHON
def get_cache_key(version, key):
    return f"{version}_{key}"

Cache Invalidation API:

PYTHON
from functools import lru_cache

@lru_cache(maxsize=None)
def cached_function(v, *args, **kwargs):
    version = v
    result = real_function(*args, **kwargs)  # Real function that may be deployed in different versions
    return result

def invalidate_caches(version):
    for old_version in get_previous_versions(version):
        clear_cache(old_version)

Cache Purge Mechanism:

PYTHON
from redis import Redis

redis_client = Redis(host='localhost', port=6379, db=0)

def clear_cache(version):
    keys_to_delete = [get_cache_key(version, key) for key in list_of_keys]
    redis_client.delete(*keys_to_delete)

Rollback Strategy:

PYTHON
def rollback_to_previous_version(current_version):
    previous_versions = get_previous_versions(current_version)
    for version in previous_versions:
        invalidate_caches(version)

#### Closing Summary
and Common Mistakes/Follow-Ups:

#### Common Mistakes

Overlooking the need for a robust invalidation mechanism.
Not considering the performance implications of cache purges, especially during peak times.
Failing to plan for a rollback strategy that minimizes downtime.

#### Follow-Ups

How would you ensure data consistency between cached and non-cached data?
What additional mechanisms could be put in place to prevent future cache pollution incidents?
How do you handle distributed caches with multiple nodes?
Depth + Language Match the Level:

This solution is tailored for a senior/strategic level, considering both technical depth and operational impact. It covers versioning, invalidation, purging, and rollback strategies, ensuring that the caching system can effectively manage potential issues.

**Comparison:**

```
Version	Key
v1	key1
v2	key2
v3	key3
```

**Code:**

```python
def get_cache_key(version, key):
    return f"{version}_{key}"

@lru_cache(maxsize=None)
def cached_function(v, *args, **kwargs):
    version = v
    result = real_function(*args, **kwargs)  # Real function that may be deployed in different versions
    return result

def invalidate_caches(version):
    for old_version in get_previous_versions(version):
        clear_cache(old_version)

def clear_cache(version):
    keys_to_delete = [get_cache_key(version, key) for key in list_of_keys]
    redis_client.delete(*keys_to_delete)

def rollback_to_previous_version(current_version):
    previous_versions = get_previous_versions(current_version)
    for version in previous_versions:
        invalidate_caches(version)
```

**Example input / output:**

```text
Input:
invalidate_caches('v3')

Output:
Cache keys for versions 'v1' and 'v2' are deleted.
```

**Data:**

```yaml
list_of_keys: ["key1", "key2", "key3"]
get_previous_versions: lambda v: [f"v{i}" for i in range(1, int(v))]
```

**Flow:**

1. Yes
2. No
3. Cache Invalidation
4. Is Current Version?
5. Purge Cache Keys for PreviousVersions
6. Continue Operating withCurrent Version
7. Notify All Nodes to PurgeCache

