---
type: "practice"
difficulty: "Expert"
---

# How do you decide the cost-optimal degree of activation checkpointing / recomputation in a large training job, trading recompute FLOPs against memory savings that let you fit bigger batches or avoid extra GPUs? Walk me through when trading compute for memory is a net cost win.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost of Parallelism Strategies](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_of_Parallelism_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deciding the optimal degree of activation checkpointing/recomputation in large training jobs involves balancing recompute FLOPs against memory savings that allow fitting larger batches or avoiding extra GPUs. The key is to trade compute for memory and evaluate whether this trade-off results in a net cost win. This decision is influenced by several factors, including batch size, GPU utilization, and overall job efficiency.

First, let's break down the problem:

Checkpointing: Storing intermediate states at regular intervals.
Recomputation: Recalculating parts of the model state from checkpoints to reduce memory usage.
FLOPs (Floating Point Operations per Second): Measuring computational intensity.

To determine if trading compute for memory is a net cost win, we need to consider both the cost implications and performance benefits. Here's a step-by-step approach:

Step 1: Understand the Current Setup

Assume you have a training job with the following characteristics:

Batch Size: B
Total Steps (Iterations): T
Current GPU Utilization: U
Memory Limit: M

The current cost is determined by the total memory usage and FLOPs consumed.

Step 2: Evaluate Checkpointing

Checkpoints can be used to reduce memory pressure. For example, if you store checkpoints every C steps:

Checkpoint Memory Usage: M/C
Recompute Steps: (T - C)

This reduces the memory footprint but increases recomputation overhead.

Step 3: Calculate Compute and Memory Trade-offs

To decide on the optimal checkpointing frequency, we need to compare the cost of recomputations with the memory savings:

Compute Cost per Recomputation: [ \text{Recompute FLOPs} = (T - C) \times B \times \text{FLOPS per Batch} ]
Memory Savings: [ \text{Memory Saved} = M - (M/C) ]
Step 4: Determine the Net Cost Win

We need to calculate the net FLOPs saved versus the additional recomputation cost:

Net Compute Savings: [ \text{Net Compute Savings} = \frac{\text{Recompute FLOPs}}{\text{Batch Size}} ]
Memory Utilization: [ \text{Effective Memory Usage} = M - (M/C) ]

If the net compute savings are greater than the memory overhead, then checkpointing is a cost win.

#### Example
Calculation

Suppose you have a training job with a batch size of 128, total steps 5000, and current GPU memory limit 16GB:

Current Memory Usage: 16GB
Desired Batch Size: 256 (higher to fit more data)
Memory Required for Bigger Batch: 32GB

If checkpoints are stored every 100 steps, the memory usage becomes 16GB/100 = 160MB, and recomputations occur (T - C) / C = (5000 - 100) / 100 = 49 times.

Recompute FLOPs: 49 \times 128 \times \text{FLOPS per Batch}
Memory Savings: 32GB - 160MB = 30.4GB
Step 5: Conclusion and Trade-offs

Evaluate if the reduced memory usage outweighs the increased recomputation cost:

If 49 \times 128 \times \text{FLOPS per Batch} < (32GB - 160MB), then checkpointing is a net win.

#### Common Mistakes
and Follow-ups
Ignoring Context: Not considering the specific job characteristics.
Over-reliance on Simple Metrics: Failing to account for the actual computational overhead of recomputations.
Ignoring Memory Overhead: Assuming memory savings are always beneficial without evaluating compute cost.

In conclusion, deciding the optimal checkpointing strategy involves a detailed analysis of both compute and memory trade-offs. It requires understanding the specific job requirements and making informed decisions based on these factors.

**Comparison:**

```
Step	Current Memory Usage (GB)	Checkpoint Interval	Recompute Steps	Net Compute Savings
5000	16	100	49	True
```

**Code:**

```python
# Example Python code to calculate net FLOPs savings
def net_flops_savings(batch_size, total_steps, checkpoint_interval):
    recompute_steps = (total_steps - checkpoint_interval) / checkpoint_interval
    recompute_flops = recompute_steps * batch_size * flops_per_batch
    memory_saved = (memory_limit / checkpoint_interval)
    effective_memory_usage = memory_limit - memory_saved
    net_savings = recompute_flops > ((memory_limit - memory_saved) * 1e9) / (batch_size * 32)
    return net_savings

# Example input parameters
batch_size = 128
total_steps = 5000
checkpoint_interval = 100
memory_limit = 16 * 1e9  # 16GB in bytes
flops_per_batch = 1e9  # example FLOPS per batch

net_savings = net_flops_savings(batch_size, total_steps, checkpoint_interval)
print("Net compute savings:", net_savings)
```

**Example input / output:**

```text
Input:
- Batch Size: 128
- Total Steps: 5000
- Checkpoint Interval: 100
- Memory Limit (GB): 16
- Desired Batch Size for Better Fit: 256
- FLOPs per Batch: 1e9

Output:
Net compute savings: True
```

**Data:**

```json
{
  "batchSize": 128,
  "totalSteps": 5000,
  "checkpointInterval": 100,
  "memoryLimitGB": 16,
  "desiredBatchSize": 256,
  "flopsPerBatch": 1e9
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. Define Job Characteristics
5. Is Memory a Bottleneck?
6. Reduce Batch Size to Fit MoreData
7. Checkpointing and RepeatedComputation Analysis
8. Calculate Net FLOPs Savings
9. Net Compute Savings >Memory Overhead?
10. Enable Checkpointing
11. Optimize Other Aspects

