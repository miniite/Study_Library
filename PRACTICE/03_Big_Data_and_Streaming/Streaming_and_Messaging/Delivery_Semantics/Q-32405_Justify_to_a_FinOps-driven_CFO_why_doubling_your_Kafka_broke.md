---
type: "practice"
difficulty: "Expert"
---

# Justify to a FinOps-driven CFO why doubling your Kafka broker footprint to enable exactly-once is worth it for the ledger pipeline but not for the clickstream pipeline. Frame the argument in dollars-per-error-class.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Exactly Once](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Delivery_Semantics/Cost_of_Exactly_Once.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To justify doubling the Kafka broker footprint for enabling exactly-once semantics in the ledger pipeline while avoiding this cost for the clickstream pipeline, we need to delve into the financial implications of errors in each context. Let's break down the reasoning step by step.

Firstly, exactly-once semantics are critical for ensuring data integrity and consistency in scenarios where precise record-keeping is paramount, such as in a ledger system. Conversely, clickstream data, while valuable, can tolerate some level of duplication without significant financial repercussions due to its nature and use cases. By doubling the Kafka broker footprint, we ensure exactly-once semantics, but this comes at a cost. We need to weigh this against the potential losses from errors.

Understanding Error Costs:

In the ledger pipeline, even one data error can result in significant financial or legal ramifications. For instance, a single incorrect transaction can lead to account discrepancies, fraud, regulatory fines, or customer disputes.
Clickstream data, on the other hand, is more forgiving of errors. Small inconsistencies may not impact user satisfaction or business outcomes as drastically.

Calculating Error Costs:

Let’s assume that each error in the ledger pipeline costs an average of $10,000 (this includes transaction fees, potential fines, and time to rectify).
For clickstream data, errors might cost only around $500 per incident due to minor inaccuracies.

Cost of Exactly-Once Semantics:

Doubling the Kafka broker footprint can increase operational costs by 40%, which is a significant financial burden.
To justify this expense, we need to ensure that the savings from reducing ledger errors more than cover the increased operational costs.

Expected Error Frequency:

For the ledger pipeline, let’s estimate an error rate of 1 in 10,000 transactions with exactly-once semantics.
Without these guarantees, this error rate might increase to 1 in 1,000 due to potential data loss or corruption.

Financial Analysis:

Expected cost per year for ledger errors (with exact-once): ( \frac{1}{10,000} \times $10,000 = $1 )
Expected cost per year for clickstream errors: ( \frac{1}{1,000} \times $500 = $0.50 )

Operational Cost Increase:

Cost of doubling Kafka brokers: ( 40% ) increase in operational costs.

Now, let’s assume the current annual operational cost for both pipelines is $100,000 each:

Additional yearly cost for exactly-once semantics (ledger): ( $100,000 + ($100,000 \times 40%) = $140,000 )
Annual savings from reducing ledger errors: ( 14 errors \times $10,000 = $140,000 )

The financial benefit of avoiding these ledger errors far outweighs the increased operational costs. The clickstream pipeline can handle occasional inaccuracies without significant financial impact, so maintaining current operational costs makes sense.

**Comparison:**

| Cost Analysis | Ledger Pipeline | Clickstream Pipeline |
|---------------|-----------------|----------------------|
| Current Op Cost per Year | $100,000 | $100,000 |
| Error Rate with Exact-Once | 0.0001 | 0.001 |
| Expected Errors per Year | 1 | 10 |
| Cost of Exact-Once Semantics (Cost Savings) | $99,999 | $500 |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Current Op Cost per Pipeline] --> B[$100,000]
    C[Doubling Broker Footprint] --> D[40% Increase in Op Costs]
    E[Exact-Once Savings Ledger] --> F[$140,000]
    G[Clickstream Error Cost] --> H[$500/year]
    I[Expected Errors Ledger] --> J[0.01%]
    K[Expected Errors Clickstream] --> L[0.1%]
```

**Example input / output:**

| Pipeline Type | Current Cost per Year | Error Rate with Exact-Once | Expected Errors per Year | Cost of Exact-Once Semantics (Cost Savings) |
|---------------|----------------------|----------------------------|-------------------------|--------------------------------------------|
| Ledger        | $100,000             | 0.0001                     | 1                       | $99,999                                   |
| Clickstream   | $100,000             | 0.001                      | 10                      | $500                                      |

**Data:**

```json
{
  "pipeline_type": ["ledger", "clickstream"],
  "current_cost_per_pipeline": "$100,000",
  "error_rate_ledger_exact_once": 0.0001,
  "error_rate_clickstream": 0.001,
  "cost_per_error_ledger": $10,000,
  "cost_per_error_clickstream": $500
}
```

**Flow:**

1. Current Op Cost
2. $100,000
3. Doubling Broker Footprint
4. 40% Increase in Op Costs
5. Exact-Once Savings Ledger
6. $140,000
7. Clickstream Error Cost
8. $500/year
9. Expected Errors Ledger
10. 0.01%
11. Expected Errors Clickstream
12. 0.1%

