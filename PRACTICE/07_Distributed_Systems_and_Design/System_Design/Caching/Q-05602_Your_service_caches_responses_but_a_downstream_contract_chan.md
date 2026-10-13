---
type: "practice"
difficulty: "Deep"
---

# Your service caches responses but a downstream contract change means the cached shape is now wrong for new clients. How do you migrate the cache without breaking either old or new clients?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Stale While Revalidate](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Stale_While_Revalidate.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this caching issue, we need to ensure that both existing and newly added clients can seamlessly interact with our service despite a change in the response shape. The key is to design a migration strategy that avoids breaking old client integrations while preparing new clients for the updated format.

The question probes how to manage state transitions between different versions of cached data without disrupting user experience or service availability. We need to make assumptions about the current cache structure, the nature of the contract change, and the capabilities of both old and new clients.

Triaging the Problem
Current Cache Structure: Assume we use an in-memory cache with a key-value store where keys are request identifiers, and values are serialized responses.
Contract Change: The response shape has changed, perhaps due to adding or removing fields.
Client Capabilities: Old clients expect the old format, while new clients should handle both formats.

#### First-Principles Solution

To migrate the cache without breaking clients, follow these steps:

Identify and Separate Old Data:

When a request comes in, check if it matches an existing cached response.
If it does, serve from cache but mark this as "old" data.

Generate New Responses for New Requests:

For new requests or requests where the old data is marked, generate the new response format.

Merge Old and New Data:

As responses are generated in the new format, update the cache with both old (compatible) and new (up-to-date) versions.
Ensure a transition period where both formats coexist.

Client Detection and Compatibility Handling:

In the response headers or body metadata, include information that helps clients determine whether they are receiving an "old" or "new" format.
Clients should be designed to handle both formats gracefully, either by falling back to old formats when necessary or adapting to new fields.

#### Implementation
Steps

Cache Key Update:

PYTHON
cache_key = f"{request_id}:{version}"

Response Handling:

For old clients:
PYTHON
def handle_request(request):
    response = get_from_cache(cache_key)
    if not response or is_old_format(response):
        # Generate new response and update cache with both old and new
        new_response = generate_new_response(request)
        updated_response = update_cache_with_both_formats(cache_key, new_response)
        return updated_response
    else:
        return response
For new clients:
PYTHON
def handle_request(request):
    response = get_from_cache(cache_key)
    if is_new_format(response):
        return response
    else:
        # Generate and return the new response
        new_response = generate_new_response(request)
        update_cache_with_both_formats(cache_key, new_response)
        return new_response

Cache Update Strategy:

PYTHON
def update_cache_with_both_formats(key, response):
    old_data = serialize(response.old_format())
    new_data = serialize(response.new_format())
    cache.set(key + "_old", old_data)
    cache.set(key + "_new", new_data)

#### Closing Summary
and Common Mistakes/Follow-Ups
Common Mistake: Ignoring the old client requirements, leading to a complete break in service.

#### Follow-Up
Questions:
How would you ensure backward compatibility if there are complex field changes?
What about performance considerations during this transition phase?
Summary

The solution involves maintaining both old and new versions of cached responses temporarily. This approach ensures that neither old nor new clients experience downtime or incorrect data, allowing for a smooth migration period.

**Comparison:**

| Cache Key | Old Data (JSON)                     | New Data (JSON)                     |
|-----------|------------------------------------|------------------------------------|
| 12345:v1_old | {"fieldA": "valueA", "fieldB": "valueB"} | {"fieldC": "valueC", "fieldD": "valueD"} |
| 12345:v1_new | -                                  | -                                  |

**Code:**

```python
def handle_request(request):
    cache_key = f"{request_id}:{version}"
    response = get_from_cache(cache_key)
    if not response or is_old_format(response):
        # Generate new response and update cache with both old and new
        new_response = generate_new_response(request)
        updated_response = update_cache_with_both_formats(cache_key, new_response)
        return updated_response
    else:
        return response

def update_cache_with_both_formats(key, response):
    old_data = serialize(response.old_format())
    new_data = serialize(response.new_format())
    cache.set(key + "_old", old_data)
    cache.set(key + "_new", new_data)

# Placeholder functions for illustration
def get_from_cache(key):
    pass

def is_old_format(response):
    pass

def generate_new_response(request):
    pass

def serialize(data):
    pass
```

**Example input / output:**

```text
- **Input:** Request with `cache_key = "12345:v1"`
- **Output:**
  - If cache contains only old data, generate new response and update cache.
  - If cache has both old and new data, return the appropriate one based on client compatibility.
```

**Data:**

```json
{
  "request_id": "12345",
  "version": "v1"
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Receive Request
5. Is Cache Available?
6. Check for Old Data
7. Handle Old Client
8. Return Old Response fromCache
9. Generate New Response,Update Cache with BothFormats
10. Return Updated Response
11. Handle New Client?
12. Generate and Return NewResponse
13. Return Old Response fromCache

