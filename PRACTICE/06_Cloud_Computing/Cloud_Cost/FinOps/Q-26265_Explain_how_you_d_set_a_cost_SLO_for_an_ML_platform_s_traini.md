---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd set a cost SLO for an ML platform's training throughput expressed as dollars-per-effective-FLOP or MFU-adjusted cost, and how that SLO changes the way teams choose model architectures, batch sizes, and hardware. What breaks when you make MFU a governed target?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Parallelism Strategies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_of_Parallelism_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Setting an SLO (Service Level Objective) on ML platform training costs in terms of dollars-per-effective-FLOP or MFU-adjusted cost can significantly influence how teams architect models, choose batch sizes, and select hardware. This approach ensures that the platform remains cost-efficient while balancing performance and resource utilization.

#### Triaging the Question

The key aspects this question is probing are:

Understanding of FinOps & Cloud Cost Governance: How to define and manage costs in a cloud environment.
Cost Modeling for ML Training: Techniques for measuring training throughput efficiency.
Impact on Model Architecture, Batch Sizes, and Hardware Choices: The trade-offs teams need to consider when optimizing for cost.

#### Assumptions
We are working with an ML platform that uses GPU or TPU resources.
The goal is to minimize costs while ensuring model performance meets requirements.
Effective-FLOP refers to the computational load adjusted by data sparsity and other factors, providing a more accurate measure of training complexity than raw FLOPs.

#### Solution
1. Defining the SLO

First, we need to define what constitutes an acceptable cost threshold for effective-FLOP. For example, if each unit of effective-FLOP costs $0.50 in a given environment, this becomes our baseline SLO.

MARKDOWN
SLO: Cost per effective-FLOP ≤ $0.50
2. Calculating Effective-FLOP and MFU

Effective-FLOP is calculated by normalizing the raw FLOPs with factors such as data sparsity, model architecture complexity, and other performance metrics.

MFU (Memory Access Factor) adjusts for memory access patterns that can impact cost significantly in GPU/Tensor processing units.

[ \text{Effective-FLOP} = \frac{\text{Raw FLOPs}}{\text{Sparsity Factor} + \text{Complexity Factor} + \dots} ]

3. Integrating SLO into the ML Workflow

To incorporate this SLO, we can use a combination of metrics and cost optimization techniques:

Batch Size Optimization: Larger batch sizes generally reduce per-sample training costs but increase memory usage.
Model Architecture Decisions: Simpler architectures are less expensive to train but might not meet performance targets. More complex architectures require more computational resources.
4. Example Workflow

Suppose a team is considering two models:

Model A: Simple, lower FLOPs but requires smaller batch sizes and thus uses fewer GPUs.
Model B: Complex, higher FLOPs but can run on larger batches, potentially reducing training time but increasing costs per effective-FLOP.

By applying the SLO, we can calculate the cost for each model: [ \text{Cost A} = \frac{\text{Total Training Time (Model A) * GPU Cost}}{\text{Effective-FLOP of Model A}} ] [ \text{Cost B} = \frac{\text{Total Training Time (Model B) * GPU Cost}}{\text{Effective-FLOP of Model B}} ]

If the SLO is met by Model A, it would be chosen over Model B.

5. Impact on Teams
Architectural Choices: Teams will favor simpler models or architectures that can run more efficiently.
Batch Size Selection: Larger batch sizes might be used to reduce per-sample costs but need careful management of memory and computational resources.
Hardware Selection: More cost-effective hardware, like older generation GPUs, may see increased use.

#### Common Mistakes
and Follow-Ups

#### Common Mistakes
Overlooking MFU Impacts: Forgetting that memory access patterns significantly affect GPU utilization can lead to suboptimal decisions.
Ignoring Sparsity Factors: Not adjusting for data sparsity in FLOP calculations can result in inaccurate cost models.

#### Follow-Ups
Impact on Performance: How does meeting the SLO impact model accuracy and performance?
Dynamic SLO Adjustment: Can the SLO be adjusted based on project requirements or budget constraints?

#### Closing Summary

Setting a cost SLO for ML platform training throughput can drive significant improvements in efficiency by guiding teams to make informed decisions about architecture, batch sizes, and hardware. However, it requires careful consideration of factors like MFU and sparsity to ensure accurate cost models.

**Comparison:**

```
Model	Raw FLOPs	Sparsity Factor	Complexity Factor	Effective-FLOP
A	10e9	2	1.5	25e6
B	15e9	3	2.0	37.5e6
```

**Code:**

```python
# Example code snippet to calculate effective-FLOP
def calculate_effective_flop(raw_flops, sparsity_factor, complexity_factor):
    return raw_flops / (sparsity_factor + complexity_factor)
```

**Example input / output:**

```text
| Model | Effective-FLOP | Total Training Time (h) | GPU Cost ($) |
|-------|---------------|------------------------|-------------|
| A     | 25e6          | 10                     | 500         |
| B     | 37.5e6        | 15                     | 900         |

Cost A: (10 * 500) / 25e6 = $0.2
Cost B: (15 * 900) / 37.5e6 = $0.4

Model A meets the SLO, so it is chosen.
```

**Data:**

```json
{
  "model_a": {
    "raw_flops": 10e9,
    "sparsity_factor": 2,
    "complexity_factor": 1.5
  },
  "model_b": {
    "raw_flops": 15e9,
    "sparsity_factor": 3,
    "complexity_factor": 2.0
  }
}
```

**Flow:**

1. Define SLO
2. Calculate Effective-FLOP
3. Integrate into Workflow
4. Optimize Model Architecture& Batch Sizes
5. Choose Hardware

