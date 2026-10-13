---
type: "practice"
difficulty: "Expert"
---

# Relevance quality varies wildly across your 30 tenant search apps and there's no shared definition of 'good.' As principal, how do you establish org-wide relevance standards and a shared judgment infrastructure without stifling domain-specific tuning?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Establishing consistent yet flexible relevance standards across multiple tenant search applications involves aligning various stakeholders while enabling domain-specific adjustments. The key is to create a robust, scalable framework that ensures quality but also allows for nuanced optimizations based on specific use cases.

First, let's define what we’re dealing with and the core goals:

Define Relevance: Establish clear metrics for relevance in search.
Stakeholder Involvement: Engage content owners, product managers, and tech leads to ensure buy-in.
Flexibility vs. Consistency: Balance global standards with localized tuning.

Here’s a step-by-step approach:

Understand the Current State:
Conduct a baseline assessment of current search systems.
Identify common pain points and successes among tenants.
Define Relevance Metrics:
Choose metrics that are widely applicable yet flexible enough to accommodate domain-specific nuances (e.g., precision, recall, click-through rate).
Establish a Governance Framework:
Form an interdisciplinary team with representatives from all domains.
Develop a set of guidelines and best practices for tuning search relevance.
Create a Shared Judgment Infrastructure:
Implement a centralized scoring system to evaluate search results against predefined criteria.
Use A/B testing to validate changes and gather feedback.

Now, let's dive into the details:

Define Relevance Metrics (TL;DR):

Precision, Recall, Click-Through Rate
Domain-specific KPIs like conversion rates or engagement metrics

Stakeholder Engagement:

Organize workshops with content owners and product managers.
Use surveys and interviews to gather input on desired outcomes.

Governance Framework (First Principles):

Develop a scoring model that incorporates common relevance factors.
Ensure the model is transparent and explainable.
Provide tools for domain-specific adjustments, such as custom weightings or field-level tuning options.

Shared Judgment Infrastructure:

Implement an A/B testing framework to compare different configurations.
Use machine learning models to provide insights on optimal parameter settings.
Create a collaborative dashboard where stakeholders can monitor and refine search results in real-time.

#### Closing Summary
By establishing clear metrics, engaging relevant stakeholders, and creating a flexible yet standardized infrastructure, we can ensure that all tenants have access to high-quality search experiences while maintaining the ability for domain-specific optimizations. This approach balances global consistency with local flexibility, avoiding the pitfalls of overly rigid or overly permissive policies.

#### Common Mistakes/Follow-Ups

Over-Rigid Standards: Failing to provide enough flexibility can stifle innovation.
Lack of Stakeholder Input: Without proper engagement, standards may not align with business goals.
Inadequate A/B Testing: Relying solely on historical data without validation can lead to suboptimal configurations.

**Comparison:**

| Metric | Definition |
|--------|------------|
| Precision | Proportion of relevant documents in the results returned by a search engine query. |
| Recall | Proportion of all relevant documents retrieved from the total number available in the system. |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Establish Baseline] --> B[Define Metrics];
    B --> C[Engage Stakeholders];
    C --> D[Develop Framework];
    D --> E[Implement Infrastructure];
    E --> F[A/B Testing];
    F --> G[Monitor & Refine];
```

**Example input / output:**

```text
| Tenant   | Current Precision | Target Precision | Tuning Parameters |
|----------|------------------|-----------------|------------------|
| Tenant A | 0.75             | 0.82            | title weight: 1.3 |
| Tenant B | 0.90             | 0.93            | description weight: 1.2 |

After applying the tuning parameters, we see an improvement in precision:
- Tenant A: From 0.75 to 0.82
- Tenant B: From 0.90 to 0.93

These adjustments were validated through A/B testing.
```

**Data:**

```json
{
  "metrics": {
    "precision": 0.85,
    "recall": 0.92,
    "click_through_rate": 3%
  },
  "tuning_options": [
    { "field": "title", "weight": 1.5 },
    { "field": "description", "weight": 1.0 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Identify Pain Points
4. Engage Stakeholders?
5. Define Metrics
6. Develop GovernanceFramework
7. Implement Infrastructure
8. A/B Testing
9. Monitor & Refine
10. Leverage Historical Data
11. Establish Standards

