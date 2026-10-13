---
type: "practice"
difficulty: "Expert"
---

# How would you cache in a service that must support tenant-level data-residency rules, so a European tenant's cached data never lands in a US-region cache node?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Caching and Service Independence](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Caching_and_Service_Independence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle caching in a service while respecting tenant-level data residency rules, especially concerning compliance requirements like GDPR (General Data Protection Regulation) and ensuring that European tenants' data never leaves their regions, we need to design a robust caching strategy. The key idea is to ensure data is cached locally within the same region where it's created or accessed by a specific tenant.

#### Triage

The question is asking for a caching solution that respects regional boundaries for storing and accessing tenant-specific data. The interviewer is probing your understanding of distributed systems, data residency compliance, cache invalidation strategies, and potential performance trade-offs. We will assume that the service involves multiple regions (e.g., US, EU) with different tenants requiring local storage.

#### First-Principles Solution
Region-Level Caching: Each region should have its own independent caching layer.
Tenant Identification: Every request must include tenant metadata to identify which cache is responsible for the data.
Cache Invalidation: A mechanism needs to handle cache invalidation across regions without causing cross-regional data leakage.

#### Implementation
Steps

Define Cache Layers by Region:

Each region will have its own Redis or Memcached instance.

Request Handling with Tenant Metadata:

Requests include tenant-specific headers that indicate which region's cache should be used for fetching and storing data.

#### Example
request header: X-Tenant-ID indicating the European tenant.

Data Flow:

When a client from an EU tenant makes a request, the system first checks if the required data exists in the local cache (EU cache).
If not found, it fetches the data from the database and stores it in both the EU and US caches but marks the EU cache as authoritative.

Cache Invalidation:

Use a message queue like RabbitMQ or Kafka to publish invalidation messages.
Upon receiving an update from the database, invalidate the cache by publishing a message with tenant-specific metadata to the appropriate region's cache nodes.
Architecture Diagram
Yes
No
No
Client
Is Tenant Data ResidencyRequest?
Fetch Local Cache
Data Found?
Database Fetch and CacheInsert
Publish Invalidate Message toRegion's Queue
Cache Invalidated
Database Fetch
Client Response
Client Response
Client Response

#### Closing Summary
and Common Mistakes
Common Mistake: Forgetting to handle edge cases where tenants from different regions access each other's data.
Mistake Follow-Up: Implement strict validation rules on request metadata to prevent cross-region cache hits.

#### Conclusion

This solution ensures that tenant-specific data remains within the correct region, adhering to compliance regulations. By leveraging distributed caching with region-specific instances and a message queue for invalidation, we can maintain both performance and regulatory requirements.

**Comparison:**

```
Tenant ID	Region
EU123	EU
```

**Code:**

```python
# Example Python code snippet for handling cache requests

def handle_request(tenant_id, request):
    # Determine the correct cache based on tenant ID
    if is_european_tenant(tenant_id):
        region = "EU"
    else:
        region = "US"

    # Check local cache
    cached_data = get_cache(region)
    if cached_data:
        return cached_data

    # Fetch data from database and cache it in both regions
    db_data = fetch_from_db(request)
    cache_data_in_both_regions(db_data, tenant_id)

    return db_data

def is_european_tenant(tenant_id):
    # Implement logic to check if the tenant ID corresponds to an EU tenant
    pass

def get_cache(region):
    # Get data from appropriate region's cache
    pass

def fetch_from_db(request):
    # Fetch data from database
    pass

def cache_data_in_both_regions(data, tenant_id):
    # Cache data in both regions with proper metadata for invalidation
    pass
```

**Example input / output:**

```text
Input:
- Tenant ID: EU123
- Request Data: { "region": "EU" }

Output:
- Cached Data from EU Cache
- Invalidation Message Published to EU Queue

Dry-run:
Request -> Check Local EU Cache -> No Hit -> Fetch from Database -> Store in Both Caches (Mark EU as Authoritative) -> Publish Invalidate Message to EU Queue.
```

**Data:**

```json
{
  "tenant_id": "EU123",
  "request_data": {
    "region": "EU"
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Client Request
5. Is Tenant Data ResidencyRequest?
6. Fetch Local Cache EU
7. Data Found?
8. Database Fetch and CacheInsert EU,US
9. Cache Invalidated
10. Client Response with EU Cache
11. Fetch Data from Database
12. Client Response with US Cache

