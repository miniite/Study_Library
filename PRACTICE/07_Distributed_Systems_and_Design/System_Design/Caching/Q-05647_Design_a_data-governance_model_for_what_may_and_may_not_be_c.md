---
type: "practice"
difficulty: "Expert"
---

# Design a data-governance model for what may and may not be cached across the org — PII, regulated fields, secrets — enforced at the platform level rather than left to each team's discipline.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Caching and Service Independence](<../../../../07_Distributed_Systems_and_Design/System_Design/Caching/Cache_Invalidation/Caching_and_Service_Independence.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to design a comprehensive data-governance model for caching that enforces strict policies around PII (Personally Identifiable Information), regulated fields, and secrets across our organization. This model will be enforced at the platform level rather than relying on individual teams to adhere to these rules. The goal is to ensure compliance with legal and regulatory requirements while optimizing performance and reducing risks.

#### Triage
Understanding the Question

The key elements of this question are:

PII, Regulated Fields, Secrets: These data types have strict handling and storage requirements.
Data Governance Model: This involves defining policies that dictate which data can be cached and how it should be managed.
Enforcement at Platform Level: The policies must be enforced uniformly across the organization to ensure consistency.

#### Assumptions
The platform supports multiple caching mechanisms (e.g., Redis, Memcached).
There are existing data classification tools in place to identify PII, regulated fields, and secrets.
Teams can query a metadata service for cached data properties before performing operations.

#### First-Principles Solution
1. Define Data Classes

First, we need to clearly define the data classes that fall under our governance rules:

PII: Any data that could be used to identify an individual (e.g., name, address, email).
Regulated Fields: Fields containing sensitive or confidential information required by regulatory bodies (e.g., financial records, health information).
Secrets: Sensitive information such as API keys, passwords, and access tokens.
2. Metadata Service

We will implement a metadata service that categorizes data based on these classes. This service can be queried to determine if cached data should adhere to specific policies:

Metadata Service
Cache Decider
3. Cache Decider

The cache decider will enforce the following rules:

Do Not Cache PII, Regulated Fields, Secrets: These data types must not be cached to ensure compliance.
Conditional Caching for Specific Use Cases: Some regulated fields or PII may still need caching if they are used in specific contexts (e.g., anonymized aggregations).
Cache Decider
Cache Policy
Do Not Cache Rule
Conditional Cache Rule
4. Caching Mechanisms

For each caching mechanism, we will implement a wrapper that adheres to the rules defined by the cache decider:

Redis Wrapper: Checks metadata before storing data.
Memcached Wrapper: Similar checks as Redis.
Cache Mechanism
Redis Wrapper
Memcached Wrapper
5. Monitoring and Compliance

Implement continuous monitoring to ensure compliance:

Audit Logs: Log all cache operations for review.
Compliance Reporting: Generate reports on the usage of cached data against policies.

#### Closing Summary

The proposed model enforces strict caching rules at the platform level, ensuring that sensitive information is not cached where it shouldn't be. By leveraging a metadata service and implementing conditional caching mechanisms, we can optimize performance while maintaining regulatory compliance.

#### Common Mistakes/Follow-Ups
Over-Caching: Teams might still cache sensitive data inadvertently if the rules are too permissive.
False Negatives: If the metadata service or cache decider misclassifies data, it could lead to non-compliance.
Performance Impact: Overly restrictive policies can impact performance; balancing security and performance is crucial.

If we had more details about existing tools and workflows within the organization, we could tailor this model further to ensure seamless integration.

**Comparison:**

```
Metadata Service Entry	PII/Regulated Fields/Secrets
12345	PII, name, email
67890	Regulated Fields, SSN, credit card number
45678	Secrets, API key, access token
```

**Code:**

```python
class CacheDecider:
    def __init__(self):
        self.metadata_service = MetadataService()

    def can_cache(self, data):
        if is_pii(data) or is_regulated_field(data) or is_secret(data):
            return False
        else:
            return True

class RedisWrapper(CacheMechanism):
    def cache_data(self, key, value):
        if self.cache_decider.can_cache(value):
            redis_client.set(key, value)
```

**Example input / output:**

```text
Request:
- Data to cache: {"name": "John Doe", "email": "johndoe@example.com"}

Response:
- Can cache result: False

Request:
- Data to cache: {"username": "john_doe", "password": "secure_password"}

Response:
- Can cache result: False
```

**Data:**

```yaml
metadata-service:
  - id: 12345
    data_type: PII
    fields: [name, email]
  - id: 67890
    data_type: Regulated Fields
    fields: [SSN, credit card number]
```

**Flow:**

1. Metadata Service
2. Cache Decider
3. Do Not Cache Rule
4. Conditional Cache Rule

