---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd A/B test two relevance configs safely in production. How do you split traffic, log per-query features, and detect a regression on a specific query segment before full rollout?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To safely A/B test two relevance configurations (relevance configs) in production, we need to ensure that the testing does not impact user experience or lead to any unintended regressions. This involves carefully splitting traffic, logging relevant metrics per query, and monitoring for issues before fully rolling out a new config.

#### Triage

The question is asking how to conduct an A/B test between two relevance configurations in a production environment while maintaining high availability and ensuring that there are no negative impacts on user experience or search performance. The key aspects include:

Traffic splitting: How to evenly distribute queries between the old and new configurations.
Logging per-query features: What specific metrics need to be logged to detect potential issues.
Detecting regressions: How to identify early signs of problems before fully switching to the new configuration.
Step-by-Step Solution
Traffic Splitting
Use a consistent hashing mechanism to distribute queries between the old and new relevance configs. This ensures that user sessions are evenly split, reducing the risk of sudden traffic surges or drops.
Logging Per-Query Features
Log relevant metrics for each query:
Query terms
Document IDs returned by both configs
Relevance scores from both configs
User click-through rates (if applicable)
Detecting Regressions
Monitor the logged data to detect any issues early:
Compare the top 10 documents returned by each config for a set of representative queries.
Track user satisfaction metrics such as average session duration, bounce rate, and number of clicks per query.
Detailed Implementation

Traffic Splitting

PYTHON
import hashlib

def split_traffic(query_id):
    hash_key = int(hashlib.md5(query_id.encode('utf-8')).hexdigest(), 16) % 2
    return "old_config" if hash_key == 0 else "new_config"

Logging Per-Query Features

JSON
{
  "query_id": "q_12345",
  "terms": ["keyword1", "keyword2"],
  "doc_ids_old": [1, 2, 3],
  "scores_old": [0.9, 0.8, 0.7],
  "doc_ids_new": [2, 3, 4],
  "scores_new": [0.85, 0.75, 0.65]
}

Detecting Regressions

Implement a monitoring system that compares the top N documents for a set of representative queries.
PYTHON
from datetime import datetime

def monitor_queries(query_ids):
    for query_id in query_ids:
        old_doc_ids = get_top_docs("old_config", query_id)
        new_doc_ids = get_top_docs("new_config", query_id)

        if old_doc_ids != new_doc_ids:
            log_regression("query_id=" + query_id, "docs_mismatch")

#### Closing Summary

By implementing a well-structured A/B testing strategy, we can safely test two relevance configurations in production. The key steps involve splitting traffic evenly, logging relevant metrics per query, and monitoring for any early signs of regression issues before fully rolling out the new configuration.

#### Common Mistakes
and Follow-Ups

#### Common Mistakes

Failing to properly randomize user sessions can lead to uneven distribution.
Not logging enough data to detect subtle changes in relevance.
Ignoring user feedback or click-through rates, which are critical for understanding actual user experience.

#### Follow-Ups

Discussing how to handle sudden spikes in traffic during testing.
Explaining the trade-offs between accuracy and precision in logging metrics.
Discussing fail-safes in case of major issues detected during monitoring.

**Comparison:**

```
Query ID	Old Config Docs	New Config Docs
q_12345	[1, 2, 3]	[2, 3, 4]
q_67890	[4, 5, 6]	[5, 6, 7]
```

**Code:**

```python
import hashlib

def split_traffic(query_id):
    hash_key = int(hashlib.md5(query_id.encode('utf-8')).hexdigest(), 16) % 2
    return "old_config" if hash_key == 0 else "new_config"

def get_top_docs(config, query_id):
    # Simulate getting top documents from Elasticsearch
    pass

def monitor_queries(query_ids):
    for query_id in query_ids:
        old_doc_ids = get_top_docs("old_config", query_id)
        new_doc_ids = get_top_docs("new_config", query_id)

        if old_doc_ids != new_doc_ids:
            log_regression("query_id=" + query_id, "docs_mismatch")
```

**Example input / output:**

```text
Input:
- Query IDs: q_12345, q_67890

Output:
- Log entries for potential regressions detected during monitoring.
```

**Data:**

```json
{
  "query_id": "q_12345",
  "terms": ["keyword1", "keyword2"],
  "doc_ids_old": [1, 2, 3],
  "scores_old": [0.9, 0.8, 0.7],
  "doc_ids_new": [2, 3, 4],
  "scores_new": [0.85, 0.75, 0.65]
}
```

**Flow:**

1. Yes
2. No
3. Regression Detected
4. No Regression
5. Query Received
6. Is Evenly Split?
7. Log Query Features
8. Fail Safe Triggers
9. Metric Comparison
10. Alert Team; Prevent Rollout
11. Roll Out New Config Gradually

