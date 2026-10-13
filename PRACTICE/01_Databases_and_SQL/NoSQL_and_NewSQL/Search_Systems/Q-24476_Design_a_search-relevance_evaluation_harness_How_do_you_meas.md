---
type: "practice"
difficulty: "Expert"
---

# Design a search-relevance evaluation harness. How do you measure NDCG/MRR offline, run A/B tests online, and prevent a relevance tuning change from silently regressing tail queries?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Relevance](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Relevance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In this scenario, we need to design an evaluation harness that can measure the effectiveness of search algorithms using metrics like Normalized Discounted Cumulative Gain (NDCG) and Mean Reciprocal Rank (MRR). Additionally, we must implement A/B testing for online user experience without negatively impacting tail queries. The goal is to ensure that any changes in relevance tuning do not regress the performance on less frequently searched terms.

#### Triage
The core of the question revolves around designing an offline evaluation mechanism and an online A/B testing strategy that minimizes risk, particularly to tail queries. We will assume a typical search engine infrastructure with Elasticsearch as the backend and a distributed system architecture.

Step-by-Step Solution

Offline Evaluation Harness for NDCG/MRR

Objective: To evaluate the relevance of search results using offline metrics.
Metrics: NDCG measures the quality of a list of ranked items, while MRR measures the rank at which the first relevant item appears in the result set.

Designing the Offline Evaluation Framework:

Data Collection and Preprocessing:
Collect historical query logs that include both queries and their corresponding click-through data.
Clean and preprocess this data to ensure it's suitable for evaluation metrics.
Evaluation Logic:
Implement a function to calculate NDCG and MRR based on the relevance scores assigned by our algorithm.
Use real query logs or synthetic data to simulate different scenarios.

A/B Testing Strategy:

Objective: To test new tuning changes in a controlled manner without impacting production users.

#### Approach
Split user traffic into control and test groups using a randomized method.
Apply the new relevance tuning only to the test group initially, while serving existing results to the control group.
Monitor the performance of both groups using real-time metrics.

Protecting Tail Queries:

Objective: To ensure that changes do not negatively impact tail queries.

#### Approach
Implement a mechanism to monitor and alert on degradation in NDCG or MRR for less frequently searched terms.
Use separate thresholds or weightings when evaluating the performance of these queries.

#### Implementation
Steps
Offline Evaluation Code Snippet:
PYTHON
import numpy as np

def ndcg_score(y_true, y_score):
    # Calculate DCG and IDCG
    dcg = sum([(2**label - 1) / np.log2(rank + 2) for rank, label in zip(range(len(y_score)), y_true)])
    idcg = dcg  # Perfect ordering
    return dcg / idcg

def mrr_score(y_true, y_score):
    # Find the reciprocal of the position at which the first relevant item appears
    reci_rank = [1.0 / (np.where(y_true == 1)[0][0] + 1) if 1 in y_true else 0 for _ in range(len(y_score))]
    return np.mean(reci_rank)
A/B Testing Framework:
PYTHON
import random

def a_b_test(query, user_id):
    # Randomly decide whether to serve the new or old relevance tuning
    if random.random() < 0.1:  # 10% chance of serving test results
        return apply_new_tuning(query)
    else:
        return apply_old_tuning(query)

def apply_new_tuning(query):
    # Apply new relevance tuning logic here
    pass

def apply_old_tuning(query):
    # Serve the existing search result
    pass
Monitoring Tail Queries:
Use a separate metric system to track NDCG and MRR for tail queries.
Set up alerts if any significant drop is observed.

#### Closing Summary
and Common Mistakes
Ensure that both offline evaluation and A/B testing are integrated seamlessly into the existing infrastructure.
Be cautious about the impact of randomization in A/B tests; ensure it doesn't introduce bias or noise.
Regularly monitor tail queries to prevent silent regressions during relevance tuning changes.

#### Common Mistakes
Overlooking the importance of monitoring tail queries, leading to unnoticed performance degradation.
Failing to implement robust alerting mechanisms that can quickly detect and address issues.
Neglecting offline validation before deploying new A/B tests, which can lead to unexpected surprises in production.

**Code:**

```
Offline evaluation code snippet
CODE
Copy
```python
# A/B testing framework
```

**Example input / output:**

```text
No input-output examples are needed here, but you can simulate test cases to verify the metrics.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Offline Evaluation
6. Is Data Available?
7. Preprocess and CalculateMetrics
8. Generate Reports
9. Collect More Data
10. A/B Testing
11. Is User Selected for Test?
12. Apply New Tuning
13. Serve Old Results

