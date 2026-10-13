---
type: "practice"
difficulty: "Intermediate"
---

# Your agent's tool registry is a distributed config service read on every step, and a config-store blip spiked agent latency fleet-wide. Explain how you'd cache the registry locally with a TTL and safe fallback so the store is never on the critical path.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this distributed systems problem, we need to ensure that our agent's tool registry (a critical configuration service) can be cached locally while maintaining safety and ensuring it’s always available even if the central store fails. The key here is implementing a caching strategy with a Time-to-Live (TTL), combined with a safe fallback mechanism.

#### Triaging the Question

The core issue we're dealing with is minimizing latency by caching a distributed configuration service, which currently introduces performance bottlenecks due to frequent network calls. We need to balance cache freshness with data consistency and reliability in case of failures.

#### First Principles Solution
Local Caching: Implement a local cache for the tool registry on each agent.
Time-to-Live (TTL): Define a TTL to ensure that cached data is refreshed periodically from the central store.
Safe Fallback Mechanism: Ensure that if the cache expires or the central store fails, the system can still access configuration data.
Detailed Steps

Local Caching:

Each agent maintains a local copy of the tool registry in memory.
This reduces the need to repeatedly fetch the same data over the network during each operation.

Time-to-Live (TTL):

Set an expiration time for cached entries so they are automatically invalidated after a certain period.
For example, if our TTL is set to 30 minutes, we refresh the cache every 15-29 minutes.

Safe Fallback Mechanism:

Implement a mechanism to check if the local cache has expired and fetch fresh data from the central store when necessary.
Ensure that the agent can handle cases where the central store is unavailable or returns an error by falling back to pre-fetched cached values or default configurations.

#### Example
Implementation
PYTHON
class ConfigService:
    def __init__(self, cache_ttl: int = 300):
        self.cache = {}
        self.ttl = cache_ttl

    def get_tool_registry(self) -> dict:
        if not self.cache or time.time() - self.cache['last_update'] > self.ttl:
            # Fetch fresh data from the central store
            self.cache = self.fetch_from_store()
        return self.cache.get('tool_registry')

    def fetch_from_store(self) -> dict:
        try:
            response = requests.get('http://central-store:8080/tool-registry')
            if response.status_code == 200:
                self.cache['last_update'] = time.time()
                return response.json()['data']
            else:
                # Handle network failures and fallback to cached data
                return self.fallback_to_cache()
        except Exception as e:
            print(f"Failed to fetch from store: {e}")
            return self.fallback_to_cache()

    def fallback_to_cache(self) -> dict:
        if 'tool_registry' in self.cache:
            return self.cache['tool_registry']
        else:
            # Return default configurations or raise an error
            return {}

# Example usage
config_service = ConfigService()
registry = config_service.get_tool_registry()

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
By implementing a local cache with TTL and a safe fallback mechanism, we can significantly reduce the latency introduced by frequent calls to the central configuration store. This approach ensures that agents have access to up-to-date configuration data while maintaining reliability in case of failures.

#### Common Mistakes
Forgetting to implement a safe fallback mechanism, leading to potential service downtime.
Setting an excessively long TTL, which could result in stale configurations being used for too long.
Not handling edge cases where the cache and central store are both unavailable.
Questions to Consider
How would you handle scenarios where the local cache is full?
What strategies can be employed to minimize network requests further?
Can you optimize the cache eviction strategy based on access patterns?

**Code:**

```python
class ConfigService:
    def __init__(self, cache_ttl: int = 300):
        self.cache = {}
        self.ttl = cache_ttl

    def get_tool_registry(self) -> dict:
        if not self.cache or time.time() - self.cache['last_update'] > self.ttl:
            # Fetch fresh data from the central store
            self.cache = self.fetch_from_store()
        return self.cache.get('tool_registry')

    def fetch_from_store(self) -> dict:
        try:
            response = requests.get('http://central-store:8080/tool-registry')
            if response.status_code == 200:
                self.cache['last_update'] = time.time()
                return response.json()['data']
            else:
                # Handle network failures and fallback to cached data
                return self.fallback_to_cache()
        except Exception as e:
            print(f"Failed to fetch from store: {e}")
            return self.fallback_to_cache()

    def fallback_to_cache(self) -> dict:
        if 'tool_registry' in self.cache:
            return self.cache['tool_registry']
        else:
            # Return default configurations or raise an error
            return {}
```

**Flow:**

1. Yes
2. No
3. Agent Fetches Registry
4. Cache Valid?
5. Return Cache
6. Fetch From Store
7. Store in Cache and Return

