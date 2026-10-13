---
type: "practice"
difficulty: "Deep"
---

# Explain how you would measure whether a data product is genuinely a product versus a repackaged table, and what governance forces the difference.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Data Product Discoverability](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Data_Product_Discoverability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Mesh architecture, the distinction between a genuine data product and a repackaged table becomes crucial. A data product should offer value beyond simple aggregation or transformation of underlying datasets; it involves delivering insights, analytics, or services that are packaged in a way that meets specific user needs. Here’s how you can measure whether a data product is genuinely a product versus a repackaged table, and the governance forces that drive this differentiation.

Firstly, let's define what we mean by a "repackaged table" versus a "genuine data product":

Repackaged Table: Simply reusing or slightly transforming existing datasets without adding significant value.
Genuine Data Product: A well-defined offering that adds value through meaningful insights, analytics, or services.

The key governance forces are centered around the principles of Data Mesh: self-service, autonomy, and trust. Here’s how you can measure and ensure these principles are being upheld:

#### Triaging the Question

The question is primarily about defining a metric to distinguish between repackaged tables and genuine data products, while also probing your understanding of the governance frameworks required to maintain quality and value in a Data Mesh environment.

#### Assumptions
Data Product Definition: A data product should be more than just a dataset; it should provide actionable insights or services.
Governance Frameworks: Governance includes roles like data stewards, metrics, SLAs, and compliance checks.

#### First Principles Solution

Value Proposition Evaluation:

Metrics for Value Addition: Develop metrics to assess the value added by each product. For example, usage metrics (API calls, queries), engagement metrics (user feedback, adoption rate), and impact metrics (business outcomes).

Process and Governance Checks:

Data Stewardship: Ensure data stewards are actively engaged in defining and maintaining products.
Governance Policies: Implement policies that enforce the creation of genuine value-added products.

Technical Indicators:

Complexity of Transformation: Measure the complexity of transformations required to derive insights from raw data (e.g., ETL job complexity, query performance).
Metadata and Documentation: Ensure proper metadata is available to describe the product's purpose and usage.

#### Example
Metrics
Usage Frequency: How often is the data product being queried or accessed?
Engagement Score: Based on user feedback surveys.
Impact Score: Measured by business outcomes (e.g., increased revenue, cost savings).
Steps to Implement Governance

Define Roles and Responsibilities:

Assign roles such as data stewards who are responsible for maintaining the quality and relevance of products.

Establish SLAs:

Set Service Level Agreements (SLAs) that define performance expectations for data products.

Regular Audits:

Conduct regular audits to ensure compliance with governance policies.

#### Closing Summary

In a Data Mesh architecture, measuring whether a data product is genuinely valuable versus merely repackaged requires defining metrics around value addition and implementing robust governance frameworks. By focusing on the principles of self-service, autonomy, and trust, organizations can ensure that their data products meet user needs and drive business outcomes.

#### Common Mistakes/Follow-Ups
Ignoring User Feedback: Failing to incorporate user feedback in product development.
Over-relying on Metrics: Relying solely on quantitative metrics without considering qualitative aspects (e.g., user engagement).
Lack of Governance: Not having clear roles, SLAs, and compliance checks can lead to poor data product quality.

#### Triaging the Question

The question is fundamentally about defining a metric for value addition and ensuring robust governance. It probes your understanding of Data Mesh principles, practical metrics, and the importance of governance in maintaining high-quality data products.

**Comparison:**

```
Metric	Description
Usage Frequency	Number of times the data product is queried
Engagement Score	Percentage of users engaging with the product (based on feedback)
Impact Score	Business outcomes derived from the use of the data product
```

**Code:**

```python
# Example Code to Define Metrics
def calculate_engagement_score(query_logs):
    # Simplified example using query logs
    total_queries = len(query_logs)
    engaged_users = set()
    for log in query_logs:
        if log['user_feedback'] == 'engaged':
            engaged_users.add(log['user_id'])

    engagement_score = (len(engaged_users) / total_queries) * 100
    return engagement_score

# Example Usage
query_logs = [
    {'user_id': 1, 'query': 'SELECT * FROM sales', 'user_feedback': 'disengaged'},
    {'user_id': 2, 'query': 'SELECT * FROM sales', 'user_feedback': 'engaged'},
    # More logs...
]
print(calculate_engagement_score(query_logs))
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Example query logs as JSON above | Engagement score calculated and returned |

**Data:**

```json
{
  "query_logs": [
    {"user_id": 1, "query": "SELECT * FROM sales", "user_feedback": "disengaged"},
    {"user_id": 2, "query": "SELECT * FROM sales", "user_feedback": "engaged"},
    // More logs...
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Data Product Initiation
6. Is Value Added?
7. Repackaged Table
8. Genuine Data Product
9. Does Governance EnsureQuality?
10. Low-Quality Product
11. Highest Value Added

