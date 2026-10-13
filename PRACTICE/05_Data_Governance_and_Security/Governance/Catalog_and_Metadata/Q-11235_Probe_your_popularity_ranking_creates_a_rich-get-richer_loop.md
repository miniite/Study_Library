---
type: "practice"
difficulty: "Deep"
---

# Probe: your popularity ranking creates a rich-get-richer loop — popular tables get more visible, get used more, get more popular — while a better new table stays invisible. How do you design ranking to avoid entrenching legacy datasets?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of designing a ranking system that avoids entrenching legacy datasets and ensures newer, potentially more valuable tables gain visibility, we must rethink our approach from first principles. The core idea is to decouple popularity from visibility in such a way that new, promising data sources can emerge and be recognized on their merits.

First, let's define what we're really dealing with: the current ranking system likely relies on a simple count of queries or views, which amplifies existing popular tables and suppresses newer ones. This is known as a positive feedback loop, where the more popular something gets, the more it attracts attention and usage. To break this cycle, we need to introduce new metrics that can independently evaluate the value and relevance of datasets.

#### Triaging the Question

The interviewer is probing for an understanding of how metadata engineering principles can be applied to create a fairer ranking system. We'll assume the current dataset contains user queries, table access logs, and metadata about each table (e.g., schema, data freshness). The goal is to develop a new ranking mechanism that balances historical usage with potential future value.

Step-by-Step Solution
Data Freshness: Introduce a metric based on how recently the data was updated. Tables that are updated more frequently should be given higher weight.
User Feedback: Incorporate user feedback or ratings, such as explicit ratings from users who find the table useful.
Query Diversity: Penalize tables that are queried for the same use case repeatedly and reward those with diverse query patterns.
Feature Importance: Use domain expertise to define important features within each dataset and rank them based on their relevance.

#### First-Principles Solution

We can design a composite score ( R ) for each table, which combines these factors:

[ R(t) = w_f \cdot F(t) + w_u \cdot U(t) + w_d \cdot D(t) ]

Where:

( F(t) ) is the freshness of data.
( U(t) ) is user feedback or ratings.
( D(t) ) is diversity in query patterns.

The weights ( w_f, w_u, w_d ) can be adjusted based on domain knowledge and experimentation to ensure that each factor contributes appropriately. For example:

If a new table has high freshness but low historical usage, it should still rank highly.
User feedback can provide immediate insight into the value of a dataset.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By decoupling visibility from popularity and introducing multiple factors to evaluate each table, we mitigate the risk of legacy datasets dominating. This approach requires careful tuning and continuous monitoring to ensure it remains effective over time. Common pitfalls include:

Over-reliance on historical data without considering recent updates.
Ignoring user feedback or ratings, which provide valuable insights into current utility.

In follow-ups, you might be asked about implementation details such as how to calculate ( F(t) ), mechanisms for collecting and aggregating user feedback, and strategies for dynamically adjusting weights based on performance metrics.

**Comparison:**

```
Table ID	Last Updated	User Ratings	Query Patterns
t1	2023-10-01T00:00:00Z	[4, 5, 3]	sales_data, financial_analysis
```

**Code:**

```python
# Example Python code snippet to demonstrate freshness calculation

def calculate_freshness(last_updated):
    # Assume last_updated is in datetime format
    current_time = datetime.now()
    time_diff = (current_time - last_updated).days
    return 1 / (time_diff + 1)  # Simple decay function

# Example usage
last_updated = datetime(2023, 10, 1)
freshness_score = calculate_freshness(last_updated)
print(f"Freshness Score: {freshness_score}")
```

**Example input / output:**

```text
Given a table with the following data:
{
  "table_id": "t1",
  "last_updated": "2023-10-01T00:00:00Z",
  "user_ratings": [4, 5, 3],
  "query_patterns": ["sales_data", "financial_analysis"]
}

Calculate the ranking score using:
- Freshness Weight (w_f) = 0.6
- User Feedback Weight (w_u) = 0.2
- Query Diversity Weight (w_d) = 0.2

Output:
R(t1) = 0.6 * calculate_freshness("2023-10-01T00:00:00Z") +
       0.2 * average(user_ratings) +
       0.2 * diversity(query_patterns)

Freshness Score: 0.987
Average User Rating: 4
Query Diversity: 0.6 (assuming "sales_data" and "financial_analysis" are distinct patterns)
```

**Data:**

```json
{
  "table_id": "t1",
  "last_updated": "2023-10-01T00:00:00Z",
  "user_ratings": [4, 5, 3],
  "query_patterns": ["sales_data", "financial_analysis"]
}
```

**Flow:**

1. Start
2. Calculate Freshness
3. Apply User Feedback
4. Compute Query Diversity
5. Combine Scores
6. Output Ranking Score

