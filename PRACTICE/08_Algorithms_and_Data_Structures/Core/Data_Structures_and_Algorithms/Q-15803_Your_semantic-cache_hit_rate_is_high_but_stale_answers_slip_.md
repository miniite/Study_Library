---
type: "practice"
difficulty: "Deep"
---

# Your semantic-cache hit rate is high but stale answers slip through after source docs change; how do you version cache entries and invalidate by document dependency, and what structure maps docs to dependent cache keys?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ LRU Cache Design](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/LRU_Cache_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of stale answers in a semantic cache due to changes in underlying documents, we need to implement a system that versions cache entries based on their dependencies and invalidates them when necessary. The core idea is to map each document to its dependent cache keys and maintain an efficient structure for tracking these dependencies.

#### Triage

The question requires us to design a caching strategy where:

Versioning: Each cache entry should have a version that reflects the latest state of the cached data.
Dependency Tracking: We need to track which documents influence which cache entries, so we can invalidate those caches when the underlying documents change.

#### Assumptions
Documents are updated frequently and inconsistently.
Caches serve queries with high frequency, making caching critical for performance.
The system must handle a large number of documents and cache entries efficiently.

#### First-Principles Solution
Document Versioning: Each document has a version that increments when the document changes.
Cache Entry Versioning: Cache entries are associated with document versions, and their validity depends on the document's current version.
Dependency Mapping: A structure maps documents to cache keys they depend upon.
Detailed Implementation

Document Store:

Each document has a unique identifier (e.g., doc_id) and a version number (version).

#### Example
schema:
JSON
{
  "id": 123,
  "content": "Sample content",
  "version": 5,
  "dependencies": [456, 789]
}

Cache Structure:

Cache entries store the document's version and a unique key.

#### Example
cache entry:
JSON
{
  "key": "cache_key_123_v5",
  "value": "cached_value"
}

Dependency Mapping:

Use a data structure like a graph or a hash map to track dependencies between documents and cache keys.

#### Example
dependency mapping (simplified):
Doc1
CacheKeyA_v1
Doc2
CacheKeyB_v1
Doc3
CacheKeyC_v1
Doc4
CacheKeyD_v1
Doc5
CacheKeyE_v2

Invalidation Logic:

When a document's version changes, invalidate all cache entries that depend on it.

#### Example
invalidation process:
PYTHON
def invalidate_caches(doc_id: int):
    # Fetch dependencies of the document from the mapping structure
    for dependency in get_dependencies(doc_id):
        # Invalidate related cache keys
        invalidate_cache_keys(dependency)

Cache Lookup:

Before serving a request, check if the cache entry is up-to-date by comparing its version with the document's current version.

#### Example
cache lookup logic (simplified):
PYTHON
def get_cache_entry(key: str) -> Union[Dict[str, Any], None]:
    # Check if the key exists in cache and is valid
    doc_id, version = parse_key(key)
    latest_doc_version = get_latest_document_version(doc_id)
    if version < latest_doc_version:
        return None  # Cache entry is stale; fetch from source
    else:
        return load_cache_entry(key)  # Return cached value

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we need a robust system that tracks document versions and their dependencies to ensure cache entries are invalidated correctly. The key steps include versioning documents, mapping documents to cache keys, using efficient data structures for dependency tracking, and implementing valid invalidation logic.

#### Common mistakes

Failing to increment the correct version numbers.
Incorrectly mapping documents to dependent cache keys.
Not efficiently managing cache entry versions and dependencies.

#### Follow-ups
could involve discussing trade-offs between performance and storage overhead, considering different caching strategies (e.g., LRU), or exploring distributed cache invalidation mechanisms.

**Comparison:**

| Document ID | Version | Dependencies |
|-------------|---------|--------------|
| 123         | 5       | [456, 789]   |
| Cache Entry Key | Value        |
|-----------------|--------------|
| cache_key_123_v5 | cached_value |

**Code:**

```python
def invalidate_caches(doc_id: int):
    # Fetch dependencies of the document from the mapping structure
    for dependency in get_dependencies(doc_id):
        # Invalidate related cache keys
        invalidate_cache_keys(dependency)

def get_cache_entry(key: str) -> Union[Dict[str, Any], None]:
    doc_id, version = parse_key(key)
    latest_doc_version = get_latest_document_version(doc_id)
    if version < latest_doc_version:
        return None  # Cache entry is stale; fetch from source
    else:
        return load_cache_entry(key)  # Return cached value

def get_dependencies(doc_id: int):
    return dependency_mapping[doc_id]

def invalidate_cache_keys(dependency_id: int):
    cache_keys_to_invalidate = [f"cache_key_{dependency_id}_{version}" for version in range(1, latest_document_version+1)]
    for key in cache_keys_to_invalidate:
        delete_from_cache(key)

def parse_key(key: str) -> Tuple[int, int]:
    parts = key.split('_')
    doc_id = int(parts[0].replace('cache_key_', ''))
    version = int(parts[-1])
    return (doc_id, version)

def load_cache_entry(key: str):
    # Load and return cache entry from storage
    pass

def get_latest_document_version(doc_id: int) -> int:
    # Fetch the latest document version from the database
    return db.get_latest_version(doc_id)
```

**Example input / output:**

```text
# Example input and output for cache entry validation
doc_id = 123
version = 4

latest_version = get_latest_document_version(doc_id)  # Assume latest version is 5
is_stale = (version < latest_version)

if is_stale:
    print("Cache entry is stale. Fetching from source.")
else:
    print("Cache entry is valid. Using cached value.")
```

**Data:**

```

{ "id": 123, "content": "Sample content", "version": 5, "dependencies": [456, 789] }

CODE
Copy
```json
{
  "key": "cache_key_123_v5",
  "value": "cached_value"
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Document Update] --> B{Increment Document Version}
4. B --> C[Update Dependency Mapping]
5. C --> D[Cache Invalidation]
6. D --> E[Cache Lookup]
7. E --> F[Fetch from Source if Stale]::is_stale
8. E --> G[Return Cached Value]::!is_stale

