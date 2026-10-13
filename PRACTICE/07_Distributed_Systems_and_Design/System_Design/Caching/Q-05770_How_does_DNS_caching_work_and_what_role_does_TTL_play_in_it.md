---
type: "practice"
difficulty: "Intermediate"
---

# How does DNS caching work and what role does TTL play in it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Caching Fundamentals](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Caching_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

DNS (Domain Name System) caching is a fundamental mechanism used to improve internet performance by reducing the number of lookups required to resolve domain names into IP addresses. At its core, DNS caching stores recent query results in memory or on disk, allowing rapid lookup for frequent requests without querying authoritative sources repeatedly. The Time-To-Live (TTL) value plays a crucial role in determining how long these cached entries remain valid before they are refreshed.

**Code:**

```python
# Example of DNS caching logic using a simple dictionary to simulate memory storage
cache = {}

def get_ip(domain):
    if domain in cache:
        print(f"Cache hit for {domain}")
        return cache[domain]
    else:
        print(f"No cache entry, querying authoritative source for {domain}...")
        ip_address = query_authoritative_source(domain)
        cache[domain] = ip_address
        return ip_address

def query_authoritative_source(domain):
    # Simulate a DNS query to an authoritative server
    import random
    ip_addresses = ["192.0.2.1", "198.51.100.1"]
    return random.choice(ip_addresses)
```

**Example input / output:**

```text
# Example dry-run of the get_ip function

>>> get_ip("example.com")
Cache hit for example.com
'192.0.2.1'

>>> get_ip("example.com")
No cache entry, querying authoritative source for example.com...
Cache hit for example.com
'198.51.100.1'
```

**Flow:**

1. Hit
2. Miss
3. User Request
4. DNS Resolver
5. Cache Check
6. Return IP Address from Cache
7. Forward to Authoritative DNSServer
8. Get IP Address and Store inCache

