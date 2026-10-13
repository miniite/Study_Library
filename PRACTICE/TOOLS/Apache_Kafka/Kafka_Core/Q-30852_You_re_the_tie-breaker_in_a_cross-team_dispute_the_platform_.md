---
type: "practice"
difficulty: "Expert"
---

# You're the tie-breaker in a cross-team dispute: the platform team wants one canonical event schema per domain, the product teams want their own topic-per-use-case. As principal, how do you adjudicate, and what principle governs when to unify versus fragment topics?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Schema Formats and Registry](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To resolve this dispute, we need to balance centralization of schemas (canonical events) against flexibility required by product teams. The key is understanding the trade-offs and applying a guiding principle that aligns with our overall architecture goals.

The core issue here revolves around whether to have unified or fragmented Kafka topics. Canonical event schemas promote consistency and ease of integration, making it easier for different parts of the organization to consume events without versioning issues. On the other hand, product teams argue for flexibility in their use cases, which can be more effectively achieved through specific topic-per-use-case models.

Firstly, let's consider a few guiding principles:

Scalability and Performance: Unified schemas might reduce the number of topics, improving performance by reducing network overhead.
Maintenance and Evolution: Centralized schema management simplifies maintenance and evolution of schemas.
Inter-team Collaboration: Canonical schemas can facilitate better communication and collaboration across teams.

To adjudicate this, we should follow a principle that emphasizes modularity while ensuring coherence. Here’s a step-by-step approach to solving the problem:

Define Domains: Identify clear, well-defined domains within your system (e.g., user management, product listing). These will serve as the basis for our canonical schemas.
Canonical Schema Design: Create one or more canonical event schemas per domain. Ensure these are flexible enough to accommodate common use cases but robust enough to handle variations without frequent breaking changes.
Fragmentation by Use Case: For each domain, define specific use cases that require unique topic-per-use-case approaches. These can be built on top of the canonical schema as necessary.

For example:

User management might have a UserEvent schema that includes events like USER_CREATED, USER_UPDATED, etc.
A product listing service could have an additional ProductUpdateRequest for specific use cases requiring more complex payloads, but still based on the Product domain’s canonical schema.

By doing this, we ensure:

Centralized Schema Management: Simplifies maintenance and reduces versioning issues.
Modular Use Cases: Teams can implement their specific requirements while adhering to overarching schemas.

#### Closing Summary
The guiding principle is to unify schemas by domain where possible, but allow for fragmented topics based on unique use cases. This approach balances centralization for simplicity and flexibility for product teams’ needs.

#### Common Mistakes/Follow-Ups

Ignoring the underlying architecture goals.
Overfragmentation leading to overly complex schema management.
Underestimating the maintenance burden of a centralized schema system.

**Comparison:**

```
Domain	Canonical Schema Fields	Canonical Events
user_management	id, email	USER_CREATED, USER_UPDATED
product_listing	id, title	PRODUCT_ADDED, PRODUCT_REMOVED
```

**Code:**

```python
# Example function to generate canonical schemas based on domain
def generate_canonical_schemas(domains):
    schemas = {}
    for domain in domains:
        schemas[domain] = {
            'fields': [],
            'events': []
        }
    return schemas

domains = ['user_management', 'product_listing']
canonical_schemas = generate_canonical_schemas(domains)
print(canonical_schemas)
```

**Example input / output:**

```text
Input:
- User management domain requires a canonical schema for USER_CREATED and USER_UPDATED events.
- Product listing service needs a more complex schema for PRODUCT_ADDED and PRODUCT_REMOVED.

Output:
- Canonical schemas for both domains are defined, but the product listing can extend these with additional fields as needed.
```

**Data:**

```json
{
  "domains": [
    { "name": "user_management", "schema": {"fields": ["id", "email"], "events": ["USER_CREATED", "USER_UPDATED"]}},
    { "name": "product_listing", "schema": {"fields": ["id", "title"], "events": ["PRODUCT_ADDED", "PRODUCT_REMOVED"]}}
  ]
}
```

**Flow:**

1. Unified
2. Fragmented
3. Define Domains
4. Unified or Fragmented?
5. Create Canonical Schemas
6. Define Specific Use Cases
7. Implement Based on CanonicalSchema
8. Extend with Additional Fieldsas Needed

