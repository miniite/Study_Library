---
type: "practice"
difficulty: "Deep"
---

# A GitHub Actions cache serving a stale database image gave a green migration run that failed against the real prod schema version. Design a cache-key strategy hashing the migration set and base-schema version that is correct yet fast, and how you'd detect a poisoned cache.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ CI Cache Correctness](<../../../../TOOLS/Git_and_GitHub/Git_Core/CI_Cache_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a cache key strategy for GitHub Actions to handle database migrations correctly while ensuring it stays up-to-date with schema changes in production, we need to create a mechanism that hashes both the migration set and the base schema version. This approach ensures that our cache is invalidated when any part of these critical components changes. Let's break down the steps required to implement this effectively.

Firstly, let’s clarify what GitHub Actions are doing here:

GitHub Actions is an automation tool that runs on a Git repository.
The cache feature allows actions to store and retrieve data (like database images) between jobs, which speeds up subsequent runs by reusing previously computed results.
However, in this scenario, the cache became stale, leading to a green migration run failing against the real production schema.

To address this issue, we need a robust cache-key strategy that accurately reflects changes in both the migration set and the base schema version. This involves:

Hashing the Migration Set: A list of all migration scripts or versions applied.
Hashing the Base Schema Version: The current state of the database schema.

The combined hash will ensure that any change, whether due to a new migration being added or an existing one being altered, invalidates the cache and triggers a fresh run against the updated schema in production.

Now, let's dive into the technical details:

Migration Set Hashing:

Maintain a list of all applied migrations.
Use a consistent hashing algorithm (e.g., SHA-256) to generate a hash based on these migration IDs or version numbers.

Base Schema Version Hashing:

Track the current schema version in the database.
Generate a hash for this schema version.

Combining Both Hashes:

Concatenate both hashes and use another hashing algorithm (e.g., SHA-256) to generate the final cache key.

To detect a poisoned cache, we need an additional mechanism that ensures our cache is always invalidated when necessary. This can be done by:

Schema Version Update Event: After every schema change in production, update a flag or marker file in Git.
Polling or Webhook-Based Invalidation:
Use GitHub Actions to periodically check for this marker file or set up webhooks to trigger cache invalidation.

Here’s how we can implement the above strategy:

PYTHON
import hashlib

def generate_cache_key(migration_set, base_schema_version):
    migration_hash = hashlib.sha256(','.join(migration_set).encode()).hexdigest()
    schema_hash = hashlib.sha256(base_schema_version.encode()).hexdigest()
    combined_hash = hashlib.sha256((migration_hash + schema_hash).encode()).hexdigest()
    return combined_hash

# Example Usage
migration_set = ['v1', 'v2']
base_schema_version = 'schema_v1.0'
cache_key = generate_cache_key(migration_set, base_schema_version)
print(cache_key)  # This will be the final cache key based on the migrations and schema version

For detecting a poisoned cache:

Create a file in Git that tracks schema changes (e.g., schema_update_marker.txt).
Use a GitHub Action to periodically check for this file. If it exists, invalidate the cache.
Alternatively, set up a webhook to notify when the schema is updated.

By following these steps, we ensure our cache key strategy is both correct and fast, preventing stale migrations from causing issues in production.

