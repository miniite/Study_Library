---
type: "practice"
difficulty: "Deep"
---

# How do you govern the cost of prompt-caching and context-reuse across an org where dozens of teams build on the same base model? Design the shared-cache architecture and the chargeback that rewards cache-friendly prompt design instead of every team paying full input-token cost.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Shared Cost Attribution](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Shared_Cost_Attribution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To govern cloud costs associated with prompt-caching and context-reuse in a large organization, we need to design a shared-cache architecture that centralizes model invocations and implements a chargeback system. This approach not only optimizes cost but also encourages teams to optimize their prompt designs for better cache hit rates.

#### Triaging the Question

The question is probing our ability to architect a scalable, efficient, and cost-effective solution for managing shared resources in a multi-team environment. The core challenges include:

Centralized Cache Management: Ensuring that multiple teams can efficiently use a single model instance without duplicating costs.
Cost Visibility and Control: Implementing mechanisms to monitor and control the usage of the model, ensuring fair distribution of costs among teams based on their cache hit rates.
Incentive Alignment: Designing financial incentives to encourage teams to optimize their prompts for better cache utilization.

#### Assumptions
Each team builds upon a common base model.
The model can be invoked multiple times with different inputs.
Teams are charged based on the number of tokenized input requests, but only pay for those that do not hit the cache.
Step-by-Step Solution
1. Architecture Design

The architecture should include:

Centralized Cache Service: A dedicated service responsible for caching model invocations and responses.
Request Router: A layer that routes incoming requests to either the cache or the model itself based on cache hits.
Chargeback System: A mechanism to track and charge teams based on their token usage.
2. Centralized Cache Service

The cache service should:

Store recent model invocations along with context information.
Implement a Least Recently Used (LRU) or time-based eviction strategy to manage memory and storage efficiently.
Provide APIs for caching new invocations, checking cache status, and purging old entries.
3. Request Router

The request router should:

Check the cache service before forwarding requests to the model.
Ensure that requests hitting the cache are not charged.
Log all requests and their outcomes (hit/miss) for chargeback tracking.
4. Chargeback System

The chargeback system should:

Track token usage per team.
Apply a flat rate or tiered pricing based on token count.
Periodically generate invoices or reports showing each team’s cost breakdown.
Detailed Implementation
Cache Service Code Example (Python)
PYTHON
class CacheService:
    def __init__(self, max_size):
        self.cache = {}
        self.max_size = max_size

    def get(self, key):
        if key in self.cache:
            return self.cache[key]
        return None

    def set(self, key, value):
        if len(self.cache) >= self.max_size:
            evicted_key = next(iter(self.cache))
            del self.cache[evicted_key]
        self.cache[key] = value

cache_service = CacheService(max_size=100)
Request Router Code Example (Python)
PYTHON
class RequestRouter:
    def __init__(self, cache_service):
        self.cache_service = cache_service

    def route_request(self, request):
        key = hash(request.prompt)  # Assuming prompt is the input identifier
        cached_response = self.cache_service.get(key)

        if cached_response:
            return cached_response
        else:
            model_response = model_instance.predict(request.prompt)
            self.cache_service.set(key, model_response)
            return model_response

router = RequestRouter(cache_service)
Chargeback System Code Example (Python)
PYTHON
class ChargebackSystem:
    def __init__(self):
        self.teams = {}

    def track_usage(self, team_id, token_count):
        if team_id not in self.teams:
            self.teams[team_id] = 0
        self.teams[team_id] += token_count

    def generate_report(self):
        report = {team: count for team, count in self.teams.items()}
        return report

chargeback_system = ChargebackSystem()

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary

By designing a centralized cache service, implementing an intelligent request router, and establishing a fair chargeback system, we can optimize cloud costs while encouraging efficient prompt usage. This approach ensures that teams are incentivized to write cache-friendly code, reducing overall expenses for the organization.

#### Common Mistakes/Follow-Ups
Over-reliance on Cache: Ensuring the cache is not overfilled and still allows fresh requests.
Fairness in Charging: Avoiding disproportionate charging of teams with high token usage but low cache hit rates.
Performance Overhead: Balancing the performance overhead introduced by cache lookups against the cost savings.

To further explore, one could:

Implement advanced caching algorithms like LRU or LFU.
Use machine learning to predict which prompts are likely to be reused and prioritize those in the cache.
Introduce tiered pricing based on historical usage patterns.

**Code:**

```python
class CacheService:
    def __init__(self, max_size):
        self.cache = {}
        self.max_size = max_size

    def get(self, key):
        if key in self.cache:
            return self.cache[key]
        return None

    def set(self, key, value):
        if len(self.cache) >= self.max_size:
            evicted_key = next(iter(self.cache))
            del self.cache[evicted_key]
        self.cache[key] = value

class RequestRouter:
    def __init__(self, cache_service):
        self.cache_service = cache_service

    def route_request(self, request):
        key = hash(request.prompt)  # Assuming prompt is the input identifier
        cached_response = self.cache_service.get(key)

        if cached_response:
            return cached_response
        else:
            model_response = model_instance.predict(request.prompt)
            self.cache_service.set(key, model_response)
            return model_response

class ChargebackSystem:
    def __init__(self):
        self.teams = {}

    def track_usage(self, team_id, token_count):
        if team_id not in self.teams:
            self.teams[team_id] = 0
        self.teams[team_id] += token_count

    def generate_report(self):
        report = {team: count for team, count in self.teams.items()}
        return report
```

**Example input / output:**

```text
Input:
- Team A sends 100 token requests, but only 20% hit the cache.
- Team B sends 50 token requests, with a 40% cache hit rate.

Output:
- Chargeback report shows that Team A incurs costs for 80 tokens (full price), while Team B pays for 30 tokens due to better cache utilization.
```

**Flow:**

1. No
2. Yes
3. User Request
4. Router Check Cache
5. Cache Hit?
6. Request Model
7. Log & Charge Token Usage
8. Return Cached Response
9. Update Cache Status

