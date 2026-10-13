---
type: "practice"
difficulty: "Deep"
---

# Probe: an interviewer says 'lineage is 90% coverage — good enough, ship it.' Where is that dangerous? Which use cases require near-100% lineage completeness and how do you communicate coverage confidence to consumers?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Lineage Graphs](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Scaling_Lineage_Graphs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a data engineer or architect says "lineage is 90% coverage — good enough, ship it," they are making a pragmatic decision based on time-to-market pressures and resource constraints. However, this approach can be dangerous in several critical use cases where near-100% lineage completeness is essential for compliance, risk management, and operational reliability.

Firstly, let's break down what we're dealing with:

Coverage: 90% coverage means that only 10% of the data assets lack complete lineage information.
Pragmatism vs. Risk: Pragmatic decisions like this can expedite projects but may introduce significant risks in terms of compliance, operational integrity, and long-term maintainability.

Now, let's dive into why near-100% lineage completeness is crucial in certain use cases:

Use Cases Requiring Near-100% Lineage Completeness
1. Compliance & Regulatory Requirements
Healthcare: HIPAA, GDPR, and other regulations often require detailed data lineage to ensure patient privacy and data accuracy.
Financial Services: Financial institutions must adhere to strict regulatory requirements like SOX (Sarbanes-Oxley) which mandate comprehensive tracking of data transformations.
2. Operational Critical Systems
Customer Relationship Management (CRM): In CRM systems, lineage is critical for understanding how customer interactions are recorded and processed.
Risk Management: Financial risk models rely heavily on accurate lineage to understand the impact of various factors and ensure model robustness.
3. Data Quality & Integrity
Data Pipelines: Ensuring data quality requires clear lineage to trace errors back to their sources and implement corrective measures.
Data Governance: Complete lineage is essential for maintaining data integrity, which in turn supports effective governance policies.
Communicating Coverage Confidence

To effectively communicate the coverage confidence to stakeholders, you need a robust strategy that balances pragmatism with risk mitigation. Here are key steps:

Identify High-Risk Areas:

Use risk assessment frameworks to identify areas where 90% coverage is insufficient.
Highlight critical components like sensitive data, regulatory systems, and operational critical applications.

Prioritize & Incremental Improvement:

Prioritize high-risk areas for full lineage capture.
Develop a roadmap for incremental improvements, setting clear milestones and timelines.

Stakeholder Communication:

Clearly articulate the risks associated with 90% coverage.
Provide evidence of the potential impact in scenarios where full lineage is critical.
Offer reassurances on ongoing efforts to enhance coverage over time.

Documentation & Reporting:

Maintain detailed documentation and regular reports on lineage completeness.
Use metrics like data freshness, accuracy, and completeness to provide tangible measures of progress.

#### Common Mistakes
Ignoring High-Risk Areas: Focusing only on 90% coverage without addressing the high-risk components can lead to significant issues down the line.
Inadequate Communication: Not effectively communicating the risks associated with incomplete lineage can result in misunderstandings and delays in addressing critical issues.
Lack of Robust Metrics: Relying solely on self-reported metrics or insufficient tracking mechanisms can undermine confidence.

#### Conclusion

While 90% coverage may be sufficient for certain projects, it is dangerous to overlook the risks associated with incomplete lineage, especially in high-stakes environments like healthcare and finance. Effective communication, prioritization, and a commitment to incremental improvement are key to managing these risks successfully.

**Comparison:**

| Data Asset | Description | Lineage Coverage |
|------------|-------------|------------------|
| order_data | Customer order data from the e-commerce system | 90% (10% missing) |
| customer_preferences | Customer preferences extracted from user interactions | 85% (15% missing) |

**Code:**

```python
# Example of lineage tracking in a data pipeline
def track_lineage(data_source, processing_steps):
    lineage = {
        "data_source": data_source,
        "processing_steps": processing_steps
    }
    return lineage

lineage_info = track_lineage("customer.csv", ["filtering", "aggregation"])
print(lineage_info)
```

**Example input / output:**

```text
Input:
- Data source: order_data.csv, customer_preferences.csv

Output:
- Lineage of order_data:
  - Source: user_profiles, Step: join
  - Source: sales_transactions, Step: filter

- Lineage of customer_preferences:
  - Source: user_engagement_events, Step: transform
```

**Data:**

```json
{
  "data_assets": [
    {
      "name": "order_data",
      "description": "Customer order data from the e-commerce system",
      "lineage": [
        {"source": "user_profiles", "step": "join"},
        {"source": "sales_transactions", "step": "filter"}
      ]
    },
    {
      "name": "customer_preferences",
      "description": "Customer preferences extracted from user interactions",
      "lineage": [
        {"source": "user_engagement_events", "step": "transform"}
      ]
    }
  ]
}
```

**Flow:**

1. Data Source
2. Join
3. Sales Transactions
4. Filter
5. User Profiles
6. Join
7. User Engagement Events
8. Transform

