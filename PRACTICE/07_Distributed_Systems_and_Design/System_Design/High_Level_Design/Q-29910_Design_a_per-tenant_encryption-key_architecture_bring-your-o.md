---
type: "practice"
difficulty: "Expert"
---

# Design a per-tenant encryption-key architecture (bring-your-own-key) for a multi-tenant SaaS where an enterprise can revoke their key to instantly cut off access — how do you architect envelope encryption key caching and the crypto-shredding delete without killing read latency?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Audit and Compliance Platforms](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Audit_and_Compliance_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To design a per-tenant encryption-key architecture in a multi-tenant SaaS, we need to ensure that each enterprise can revoke its own key at any time while maintaining low read latency. This involves implementing envelope encryption with key caching and smart eviction strategies, along with a mechanism for crypto-shredding the keys upon revocation. The design balances security, performance, and reliability.

#### Triaging the Question

The core of this question revolves around designing a robust per-tenant encryption system that can handle key revocation efficiently while minimizing read latency. Key aspects include:

Key Management: How to securely store and manage tenant keys.
Caching Strategy: Efficient caching mechanisms for frequently accessed keys.
Revocation Mechanism: Swiftly revoking a key without impacting active operations.
Crypto-Shredding: Ensuring data remains unreadable post-revocation.

#### Assumptions
The system supports multiple tenants, each with their own encryption keys.
Tenant keys can be revoked at any time.
We need to ensure minimal read latency during key revocation.

#### First-Principles Solution
1. Key Storage and Management:
Use a secure key management service (KMS) like AWS KMS, Azure Key Vault, or Google Cloud KMS to store and manage tenant keys.
Each tenant will have its own set of encryption and decryption keys stored securely.
2. Envelope Encryption:
Implement envelope encryption where the actual data is encrypted with a per-tenant key (outer layer), and then this outer layer is further encrypted using a master key (inner layer).
This allows for fine-grained access control at both tenant and data levels.
3. Key Caching Strategy:
Implement an in-memory caching mechanism to store frequently accessed keys.
Use a cache like Redis or Memcached, which can be configured with eviction policies such as LRU (Least Recently Used) to manage key storage efficiently.
Cache entries should have an expiration time for the inner master key and an indefinite expiry for tenant keys.
4. Revocation Mechanism:
On revoking a key, ensure immediate update in the cache metadata to mark the key as revoked.
For tenants who still need access to data encrypted with the old key, use a delayed eviction strategy where the key remains accessible until all data is read or refreshed from storage.
5. Crypto-Shredding:
When revoking a key, perform crypto-shredding by overwriting the key in memory and on disk.
Use techniques like bitwise XOR with a random value to make it impossible to recover the original key.
Log revocation events and maintain audit trails for compliance.
6. Handling Key Revocation Without Killing Read Latency:
For read operations, ensure that even if a key is revoked, previously cached data can still be decrypted using the cache entry's metadata.
Implement a delayed expiry mechanism where the key remains accessible until all cached entries are cleared or refreshed.
7. Failover and Backup Strategies:
Ensure redundancy in KMS to avoid single points of failure.
Maintain backups of keys, but only for historical purposes, not active use.

#### Closing Summary

The design ensures that tenant keys can be revoked with minimal impact on read latency by leveraging caching strategies, delayed evictions, and crypto-shredding techniques. The system remains resilient and secure while maintaining optimal performance.

#### Common Mistakes/Follow-Ups
Ignoring Eviction Policies: Not having a well-defined eviction policy for key caches can lead to memory leaks.
Insufficient Backup Strategies: Failing to maintain proper backups or not implementing failover mechanisms can result in data loss.
Improper Crypto-Shredding: Overlooking the importance of securely wiping keys during revocation can expose sensitive data.

**Comparison:**

| Tenant ID | Action | Timestamp |
|-----------|--------|-----------|
| 1234567890 | Active | 2023-09-30T12:00:00Z |
| 1234567890 | Revoked | 2023-10-01T12:00:00Z |

**Code:**

```python
# Example Cache Eviction Policy using Redis
from redis import Redis

cache = Redis()

def cache_key(key_id):
    return f"tenant_{key_id}_data"

@cache.ttl(cache_key(tenant_key_id))
def get_encrypted_data(tenant_key_id, data_id):
    # Get cached encrypted data
    encrypted_data = cache.get(cache_key(tenant_key_id))
    if not encrypted_data:
        # Fetch from storage and store in cache with TTL
        stored_data = fetch_from_storage(data_id)
        cache.set(cache_key(tenant_key_id), stored_data, ex=600)  # Cache for 10 minutes
    return encrypted_data

@cache.evict(cache_key(tenant_key_id))
def revoke_tenant_key(tenant_key_id):
    # Mark key as revoked in metadata
    cache.metadata.set_status(tenant_key_id, "revoked")
    # Perform crypto-shredding of the key
    shred_key(tenant_key_id)
```

**Example input / output:**

```text
Input:
tenant_key_id = '1234567890'
data_id = '1234567890'

Process:
1. Cache key lookup for tenant_1234567890_data_1234567890
2. Check cache status and expiry time.
3. If active, return cached data; else fetch from storage.

Output:
Encrypted Data (if active in cache)
```

**Data:**

```json
{
  "tenant_keys": {
    "tenant_1234567890": {
      "status": "active",
      "inner_master_key": "<encrypted_inner_master_key>",
      "outer_tenant_key": "<encrypted_outer_tenant_key>"
    }
  },
  "cache_metadata": {
    "tenant_1234567890_data_1234567890": {
      "status": "active",
      "expiry_time": "2023-10-01T12:00:00Z"
    }
  },
  "revocation_log": [
    {
      "tenant_key_id": "1234567890",
      "timestamp": "2023-09-30T12:00:00Z",
      "status": "revoked"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Tenant Key Request
4. Is Tenant Key Active?
5. Retrieve Data from Cache
6. Return Encrypted Data
7. Fetch Data from Storage
8. Store in Cache with Expiry

