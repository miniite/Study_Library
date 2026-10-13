---
type: "practice"
difficulty: "Expert"
---

# Your team wants to move from a hosted embedding API to a self-hosted embedding model to cut per-embedding cost, but you embed billions of items. Build the honest TCO including GPU serving, re-embedding on model updates, and the ops burden, and find the volume where self-hosting wins.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ TCO of Self-managed Platforms](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/TCO_of_Self-managed_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine when self-hosting an embedding model would be more cost-effective than using a hosted API, we need to calculate the Total Cost of Ownership (TCO) considering factors like GPU serving costs, re-embedding expenses during model updates, and operational burden. We'll start by breaking down each component from first principles.

#### Triaging the Question

The key aspects this problem probes are:

Understanding Cloud vs. Self-Hosting Costs: Comparing the financial implications of using a hosted service versus managing infrastructure ourselves.
Model Update Considerations: Handling the retraining and reembedding processes post-model update, which can significantly impact costs.
Operational Overhead: Estimating the operational burden associated with self-hosting.

#### Assumptions
We will use AWS for both hosted services and self-hosted infrastructure.
The embedding model is in TensorFlow or PyTorch.
Model updates occur every 6 months.
The per-item cost of using a hosted API is $0.01, which we aim to reduce by self-hosting.

#### First Principles Solution
Step 1: Cost Breakdown for Hosted Service

For the hosted service: [ \text{Total Cost}_{\text{Hosted}} = n \times (\text{Cost per Embedding}) + (n \times f(\text{Model Updates})) ] Where ( n ) is the number of embeddings, and ( f(\text{Model Updates}) ) accounts for any additional costs during model updates.

Step 2: Cost Breakdown for Self-Hosting

For self-hosting:

Initial Setup Costs:
GPU Instances (Assume 4x V100 GPUs): Initial setup of infrastructure.
GPU Serving Costs:
Running the embedding service on these GPUs.
Model Update Costs:
Retraining and reembedding during updates.

[ \text{Total Cost}_{\text{Self-Hosting}} = (Initial Setup) + (\text{Cost per Embedding} \times n) + ((n/2 \times \text{Update Interval}) \times \text{Model Update Costs}) ]

Step 3: Operational Burden

Operational burden includes managing the infrastructure, monitoring performance, and ensuring data security. This can be estimated based on the number of embeddings processed.

Detailed Calculation Example

Assume:

Number of embeddings ( n = 10^9 ).
Cost per embedding for hosted service is $0.01.
Initial setup cost for self-hosting: $5,000 (including GPUs and other infrastructure).
Annual GPU serving cost: $20,000.
Model update interval: every 6 months.
Model update costs (retraining + reembedding): $4,000 per run.
Hosted Service Cost

[ \text{Total Cost}_{\text{Hosted}} = 10^9 \times 0.01 + 5 \times 10^8 \times 0.01 = $10,000,000 + $500,000 = $10,500,000 ]

Self-Hosting Cost

[ \text{Initial Setup} = $5,000 ] [ \text{Annual GPU Serving Costs} = $20,000 ] [ \text{Number of Updates per Year} = 2 \times (10^9 / 2 \times 10^8) = 10 ] [ \text{Model Update Costs per Year} = 10 \times 4,000 = $40,000 ]

Thus, [ \text{Total Cost}_{\text{Self-Hosting}} = 5,000 + (20,000 \times 1) + (40,000 \times 1) = $65,000 ]

#### Closing Summary

The TCO for self-hosting is significantly lower than the hosted service cost when embedding a billion items. The break-even point is achieved quickly due to the substantial reduction in per-embedding costs.

#### Common Mistakes
and Follow-Ups
Ignoring Model Update Costs: Failing to account for the retraining and reembedding expenses during model updates.
Overlooking Initial Setup Costs: Not considering one-time setup costs that might be high upfront but reduce long-term costs.
Focusing Only on Embeddings: Not factoring in additional operational overhead like GPU management, monitoring, and security.

**Comparison:**

| Cost Type         | Hosted Service              | Self-Hosting                 |
|------------------|----------------------------|------------------------------|
| Initial Setup     | -                           | \$5,000                      |
| Annual GPU Serving| \$20,000                    | \$20,000                     |
| Model Updates     | \$(n/2) * 0.01              | \$40,000 (2x updates/year)   |
| Total Cost        | \$10,500,000                | \$65,000                     |

**Code:**

```python
# Example Python code for cost calculations

def hosted_cost(n):
    return n * 0.01 + (n / 2) * 0.01

def self_hosting_cost(n):
    initial_setup = 5000
    annual_gpu_serving = 20000
    model_update_costs = 4000 * (n // (2e8))
    return initial_setup + annual_gpu_serving + model_update_costs

# Example values
n_embeddings = 10**9
hosted_cost(n_embeddings)
self_hosting_cost(n_embeddings)
```

**Example input / output:**

| Inputs                | Output (Hosted)            | Output (Self-Hosting)       |
|-----------------------|----------------------------|-----------------------------|
| n_embeddings          | $10,500,000                | $65,000                     |

**Data:**

```json
{
  "embeddings": 1000000000,
  "cost_per_embedding_hosted": 0.01,
  "initial_setup_self_hosting": 5000,
  "annual_gpu_serving_cost": 20000,
  "model_update_costs": 4000
}
```

**Flow:**

1. Initial Setup
2. Annual GPU Serving
3. Model Updates
4. Total Cost
5. Hosted Service Total Cost
6. Self-Hosting Total Cost

