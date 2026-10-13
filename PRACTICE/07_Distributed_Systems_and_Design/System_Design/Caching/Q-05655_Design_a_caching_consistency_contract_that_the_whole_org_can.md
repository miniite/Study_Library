---
type: "practice"
difficulty: "Expert"
---

# Design a caching consistency contract that the whole org can reason about: a small set of named freshness tiers with defined guarantees, so product and legal can pick a tier without understanding the internals.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Caching and Service Independence](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Caching_and_Service_Independence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary Overview

In this caching design exercise, we aim to create a simple yet robust system that allows the entire organization to understand and reason about cache freshness levels. We will define a small set of named tiers with clear guarantees, ensuring that product managers and lawyers can choose an appropriate tier without needing to understand the technical details.

#### Triaging the Question

The core challenge here is designing a caching consistency contract that abstracts away the complexities of caching mechanisms from business stakeholders. Specifically:

Stakeholders: Product and legal teams.
Requirements: Named freshness tiers with clear guarantees.

#### Assumptions
Stakeholders will understand general concepts like "fresh" vs. "stale."
The system must support at least three freshness levels: FRESH, STALE, and EXPIRED.
Goal: Ensure that all team members can make informed decisions about cache usage based on predefined guarantees.
Step-by-Step First Principles Solution

Define Freshness Tiers:

FRESH: Data is fresh from the source, with a strict maximum age limit.
STALE: Data is older than the maximum freshness period but not outdated enough to trigger an update request.
EXPIRED: Data has expired and should be considered invalid.

Establish Clear Guarantees:

For FRESH data, ensure it does not exceed a specified age (e.g., 1 second).
For STALE data, allow it to be cached for up to 30 seconds after the FRESH period.
For EXPIRED data, invalidate it immediately.

Integrate with Existing Systems:

Ensure compatibility with existing caching systems (e.g., Redis, Memcached).
Define APIs for setting and retrieving cache values based on freshness levels.

Documentation and Training:

Provide documentation that explains the purpose of each tier.
Conduct training sessions to ensure all stakeholders understand the implications of using different tiers.

#### Closing Summary

By defining a small set of named freshness tiers with clear guarantees, we can empower product and legal teams to make informed decisions about cache usage. This approach ensures that business stakeholders do not need deep technical knowledge, while still providing robust and reliable caching mechanisms.

#### Common Mistakes/Follow-Ups
Overcomplication: Defining too many tiers or overly complex guarantees.
Inadequate Training: Not adequately educating all stakeholders on the use of different tiers.
Lack of Flexibility: Failing to provide enough options for different business needs.

Would you like me to elaborate further on any aspect, such as specific implementation details, examples, or more detailed documentation?

**Comparison:**

```
Tier	Max Age (Seconds)	Grace Period (Seconds)	Action on Expiration
FRESH	1	N/A	N/A
STALE	N/A	30	N/A
EXPIRED	N/A	N/A	Invalidate Immediately
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[CacheFreshnessTier] --> B[FRESH];
    B --> C{Age Limit};
    C --> D[1 second];

    E[CacheFreshnessTier] --> F[STALE];
    F --> G{Age Range};
    G --> H[30 seconds after FRESH];

    I[CacheFreshnessTier] --> J[EXPIRED];
    J --> K[Immediate Invalidiation];
```

**Example input / output:**

```text
# Example input for cache freshness tier selection
request = {
    "resource": "/api/v1/user/profile",
    "tier": "FRESH"
}

# Expected output based on the defined tiers
response = {
    "cache_ttl_seconds": 1,
    "cache_action_required": False
}
```

**Data:**

```yaml
freshness_tiers:
  - name: FRESH
    max_age_seconds: 1

  - name: STALE
    grace_period_seconds: 30

  - name: EXPIRED
    action_on_expiration: invalidate_immediately
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Cache Request
6. Is Cache Fresh?
7. Return Cache
8. Fetch from Database
9. Set Cache with Tier FRESH
10. Return Data with TTL 1 second
11. Cache Expiry Check
12. Is Cache Expired?
13. Invalidate Cache
14. Return Cache

