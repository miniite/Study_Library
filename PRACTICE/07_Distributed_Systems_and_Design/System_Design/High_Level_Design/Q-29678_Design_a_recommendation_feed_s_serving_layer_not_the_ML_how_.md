---
type: "practice"
difficulty: "Deep"
---

# Design a recommendation feed's serving layer (not the ML) — how do you fetch candidates, apply real-time filters, rank, and cache to hit a 100ms budget?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a recommendation feed’s serving layer that meets a 100ms budget, we need to ensure efficient fetching of candidates, applying real-time filters, ranking, and caching. The goal is to balance performance with the complexity of the system while ensuring high availability and scalability.

#### Triage

The question asks us to design a solution for a recommendation feed’s serving layer, focusing on fetching candidates, applying real-time filters, ranking, and caching to meet a strict latency requirement of 100ms. The interviewer is probing your understanding of system architecture, distributed systems concepts, cache invalidation strategies, and optimization techniques.

#### Assumptions
We have access to an ML model that provides candidate recommendations.
The backend can handle tens of thousands of requests per second.
Real-time filters include basic user preferences, location, and time-of-day considerations.
The recommendation feed is part of a larger application with multiple other services.

#### First Principles Solution

Fetching Candidates:

Use an HTTP API endpoint to fetch candidates based on the user’s profile and context.
Assume the ML model runs asynchronously, providing recommendations in batches or as streams depending on latency requirements.

Applying Real-Time Filters:

Implement filters at both the application layer and database query level.

#### Example
filters include geographic proximity, time-of-day relevance, and user-specific preferences.

Ranking:

Apply ranking logic to sort candidates based on a combination of ML scores and real-time filters.
Use a scoring function that combines multiple factors like relevance score, freshness, and recency.

Caching:

Cache frequently accessed data using in-memory caching (e.g., Redis) for high-speed lookups.
Implement cache invalidation strategies to ensure stale data does not affect user experience.

Latency Optimization:

Optimize database queries and reduce the number of database round-trips.
Use load balancers and distributed systems techniques to distribute traffic evenly across servers.
Detailed Solution

Fetching Candidates:

User Request
API Endpoint
Database Query for InitialCandidates
Candidate Filter Service
Ranked Results Cache
API Response to User
The API endpoint receives the user's request and queries the database for initial candidates.
Candidate filter service applies real-time filters (e.g., location, time-of-day).
Ranked results are cached in an in-memory store like Redis.

Applying Real-Time Filters:

User Request
Filter Service
Database Query for FilteredCandidates
Ranked Results Cache
API Response to User
The filter service runs on each request and applies real-time filters.
These filtered candidates are then ranked using the ML scores.

Ranking:

PYTHON
def rank_candidates(candidates, user_profile):
    # Example scoring function combining ML score and real-time filters
    return sorted(
        candidates,
        key=lambda c: (c.ml_score * 0.8 + c.relevance_score * 0.2) * get_relevancy_weight(user_profile),
        reverse=True
    )

Caching:

User Request
Cached Candidates
Database Query (if cache miss)
Ranked Results Cache
API Response to User
Cache Invalidate Event
Cached Candidates
Use Redis for caching ranked results.
Implement a TTL mechanism and event-driven invalidation to keep the cache fresh.

#### Closing Summary

To meet the 100ms budget, we need an efficient architecture that fetches candidates quickly, applies real-time filters accurately, ranks them appropriately, and caches frequently accessed data. We use Redis for caching ranked results and implement robust invalidation strategies. The system is designed to handle high concurrency with distributed caching and load balancing.

#### Common Mistakes/Follow-Ups
Forgetting to consider the impact of cache invalidation on performance.
Overlooking the need for efficient database queries, which can lead to slow fetch times.
Not prioritizing user experience by ensuring timely updates to the recommendation feed.

**Comparison:**

| Candidate | ML Score | Relevance Score | Final Score |
|-----------|----------|-----------------|-------------|
| 1         | 0.95     | 0.8             | 0.864       |
| 2         | 0.75     | 0.6             | 0.576       |

**Code:**

```python
def rank_candidates(candidates, user_profile):
    # Example scoring function combining ML score and real-time filters
    return sorted(
        candidates,
        key=lambda c: (c.ml_score * 0.8 + c.relevance_score * 0.2) * get_relevancy_weight(user_profile),
        reverse=True
    )
```

**Example input / output:**

```text
Input:
User Profile: {"location": "New York", "time_of_day": "evening"}
Request: GET /recommendations

Output:
{
  "ranked_candidates": [
    {"id": 1, "score": 0.864, "metadata": {...}},
    {"id": 2, "score": 0.576, "metadata": {...}}
  ]
}
```

**Data:**

```json
{
  "candidates": [
    {"id": 1, "ml_score": 0.95, "relevance_score": 0.8},
    {"id": 2, "ml_score": 0.75, "relevance_score": 0.6}
  ]
}
```

**Flow:**

1. User Request
2. API Endpoint
3. Database Query for InitialCandidates
4. Candidate Filter Service
5. Ranked Results Cache
6. API Response to User

