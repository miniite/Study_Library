---
type: "practice"
difficulty: "Intermediate"
---

# You manage a fleet where deploys are gated by a global feature-flag service that every pod reads. Explain how you'd keep flag reads from becoming a correlated SPOF and how a bad flag flip should be blast-radius-scoped.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Config as Single Point of Failure](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Shared_Config_as_Single_Point_of_Failure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To manage a fleet where deploys are gated by a global feature-flag service, we need to ensure that each pod reads its flags independently to avoid creating a single point of failure (SPOF). Additionally, any bad flag flip should be isolated so as not to affect other parts of the system. The key idea is to decouple the flag reads from the global service and implement a local caching mechanism in each pod. Let's break down this solution step by step.

First, we need to understand that a correlated SPOF can occur if all pods rely on a single central feature-flag service. If this service fails or experiences high latency, it could impact the entire fleet. To mitigate this risk, we'll implement a caching strategy where each pod maintains its own copy of the flags and updates them periodically from the global service.

#### Triaging the Question

The question is asking for ways to ensure that flag reads do not become an SPOF and how to limit the blast radius if a bad flag flip occurs. We need to consider the reliability, latency, and failure modes of our system components.

#### Assumptions
The global feature-flag service has high availability but may experience occasional failures or network issues.
Pods communicate with the global service over a network that can have variable latency.

#### First-Principles Solution
Step 1: Implement Local Caching in Each Pod

Each pod will maintain its own local cache of feature flags. This local cache will be periodically refreshed from the global feature-flag service to ensure it remains up-to-date. By doing this, each pod can make independent decisions based on its cached data even if the global service is unavailable.

Step 2: Asynchronous Updates with Exponential Backoff

When a pod needs to refresh its flags, it should do so asynchronously and use an exponential backoff strategy to handle retries. This approach prevents overwhelming the global feature-flag service during high traffic periods or failures.

Step 3: Circuit Breaker Pattern

Implement a circuit breaker pattern in each pod’s communication with the global feature-flag service. If the service is unresponsive for a certain period, the pod should fall back to using its cached flags and retry later without overwhelming the system.

Step 4: Isolate Failures

If a bad flag flip occurs (e.g., an incorrect or harmful flag value), it should be isolated so that only affected parts of the fleet are impacted. This can be achieved by versioning feature flags and ensuring that changes are rolled out gradually, allowing time to observe potential issues before fully deploying them.

#### Closing Summary

By implementing local caching with periodic updates from a global service, using an asynchronous refresh strategy, and applying circuit breakers, we can effectively manage the risk of correlated SPOFs. Additionally, versioning flags and gradual rollouts help contain any negative impacts should a bad flag flip occur.

#### Common Mistakes/Follow-Ups
Not Implementing Local Caching: Relying solely on the global service for every request increases the likelihood of an SPOF.
Synchronous Updates: Using synchronous updates can lead to cascading failures and increased load on the global service during high traffic periods.
Ignoring Circuit Breakers: Failing to implement circuit breakers means that a single point of failure in the global service could bring down multiple pods.
Further Discussion Points
Latency Sensitivity: Depending on the application, the latency from the feature-flag service might be critical, and we need to ensure it is kept minimal.

#### Cons
istency vs. Availability Trade-offs: In some cases, consistency of flag values across all pods might be more important than availability. We should discuss how to balance these requirements.

**Comparison:**

| Flag Name | Value | Last Updated Time |
|-----------|-------|------------------|
| FLAG_A    | true  | 2023-10-01T14:00:00Z |
| FLAG_B    | false | 2023-10-01T15:00:00Z |

**Code:**

```python
# Example Python code for local caching with exponential backoff and circuit breaker

import time
from threading import Lock
from random import expovariate

class FeatureFlagManager:
    def __init__(self, global_service_url):
        self.cache = {}
        self.lock = Lock()
        self.service_url = global_service_url
        self.last_refresh_time = 0
        self.refresh_interval = 10  # initial refresh interval in seconds

    def get_flag(self, flag_name):
        with self.lock:
            if time.time() - self.last_refresh_time > self.refresh_interval:
                self._refresh_flags()
            return self.cache.get(flag_name)

    def _refresh_flags(self):
        try:
            response = requests.get(self.service_url)
            flags = response.json()['flags']
            for flag in flags:
                self.cache[flag['name']] = flag['value']
            self.last_refresh_time = time.time()
        except Exception as e:
            # Handle exceptions and use cached values
            pass

    def _should_update(self):
        return random.random() < expovariate(1 / 30)  # exponential backoff with average delay of 30s
```

**Example input / output:**

```text
Input:
- Feature flag request from pod A for FLAG_A

Output (local cache up-to-date):
- FLAG_A value is returned from local cache

Input:
- Feature flag request from pod B for FLAG_B during service unavailability

Output (using cached value):
- FLAG_B value from last known good state in the cache
```

**Data:**

```json
{
  "flags": [
    {"name": "FLAG_A", "value": true},
    {"name": "FLAG_B", "value": false}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Pod
6. Is cache up-to-date?
7. Return cached flag value
8. Fetch new flags from globalservice
9. Is service available?
10. Update local cache with newvalues
11. Use cached values, retry later

