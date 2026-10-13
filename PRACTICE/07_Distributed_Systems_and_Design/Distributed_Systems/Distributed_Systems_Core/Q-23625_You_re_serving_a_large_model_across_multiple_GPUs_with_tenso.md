---
type: "practice"
difficulty: "Expert"
---

# You're serving a large model across multiple GPUs with tensor parallelism behind a low-latency API. Explain the added inter-GPU communication on every token and how it changes your latency budget versus a single-GPU model.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Workload Isolation on Shared Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Workload_Isolation_on_Shared_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the added inter-GPU communication in a distributed setup with tensor parallelism, let's first outline what we're dealing with: serving a large language model using multiple GPUs to achieve faster inference times and lower latency. The key challenge is managing the additional communication overhead that comes from distributing model states across these GPUs.

#### Triaging the Question

The question probes your understanding of how distributed computing impacts real-time performance, particularly focusing on the trade-offs between inter-GPU communication costs and overall latency. We need to consider several factors: the architecture of the distributed system, the communication patterns, and how these impact the model's throughput and response time.

#### First-Principles Solution

In a single-GPU setup, each token is processed sequentially within that GPU without needing to communicate with others. However, in a multi-GPU setting, each token can be split across multiple GPUs for parallel processing (tensor parallelism). This requires additional inter-GPU communication, which impacts the overall latency and throughput.

Detailed Explanation
Token Parallelism: When serving a model, we often break down the input sequence into tokens that are processed in parallel across multiple GPUs. For instance, if you have a batch size of 8 and 4 GPUs, each token is split among these GPUs.
Inter-GPU Communication: Each GPU computes its part of the tensor independently but needs to share intermediate results with other GPUs to complete the model's forward pass. This inter-GPU communication introduces latency because it involves data transfer across the network interface between GPUs.
Latency Budget: The latency budget for a real-time API is critical, as users expect quick responses. With single-GPU models, this budget is simpler to manage since all computations are contained within one GPU. In contrast, in a distributed setup, we must account for additional latency due to inter-GPU communication.

#### Example Scenario

#### Cons
ider serving a 12-layer transformer model with tensor parallelism across 4 GPUs. Each token is processed by different layers spread across these GPUs. For example:

Layer 0 and 3 on GPU 1,
Layers 1 and 4 on GPU 2,
Layers 2, 5, 8, 9, and 12 on GPU 3,
Layers 6, 7, 10, and 11 on GPU 4.

Each layer's output needs to be aggregated from all relevant GPUs before moving forward. For instance, the intermediate results from Layer 3 (on GPU 1) need to be communicated to GPUs handling subsequent layers (Layers 4, 5, 6). This inter-layer communication adds latency that wasn't present in a single-GPU setup.

Impact on Latency

The added inter-GPU communication increases the total latency budget. For example, if each intra-GPU computation takes 10 microseconds and each cross-GPU communication takes 20 microseconds (due to network latencies), then every token will experience an additional delay from these communications. This can be substantial for real-time applications where low-latency is crucial.

#### Closing Summary

In summary, tensor parallelism across multiple GPUs introduces inter-GPU communication that affects the overall latency budget. While it allows for faster processing of each token by leveraging parallelism, this comes at the cost of additional network overhead. Managing this trade-off effectively requires careful architectural design and optimization techniques to balance throughput with real-time constraints.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the impact of inter-GPU communication on latency.

#### Follow-Up
Discussing strategies like model parallelism, pipeline parallelism, or using efficient communication protocols (like NCCL) to mitigate these costs.

**Comparison:**

```
Token	GPU 1 (layers)	Inter-GPU Comm.	GPU 2 (layers)
hello	0, 3	-> Layers 4,5
world		-> Layers 10,8	6, 7
Token	GPU 3 (layers)	Inter-GPU Comm.	GPU 4 (layers)
hello		-> Layers 2,5
world	6, 7	-> Layers 11	8, 9
```

**Code:**

```python
# Example code for tensor parallelism setup
import torch

class DistributedModel(torch.nn.Module):
    def __init__(self, model):
        super(DistributedModel, self).__init__()
        self.model = model
        self.world_size = 4  # Number of GPUs
        self.rank = torch.distributed.get_rank()

    def forward(self, x):
        outputs = []
        for i in range(self.world_size):
            if i != self.rank:
                x = x.clone()  # Simulate sending the tensor to a different GPU
            output = self.model(x)
            outputs.append(output)
        return torch.cat(outputs)  # Combine results from all GPUs
```

**Example input / output:**

```text
Input:
- Tokens: "hello" and "world"

Output (Simplified):
- Token "hello": GPU1 processes layers 0 & 3; GPU2 processes layers 1 & 4.
- Token "world": GPU3 processes layers 6, 7, 10, 11; GPU4 processes layers 2, 5, 8, 9.

Inter-GPU Communication:
- Layer 3 (GPU1) -> Layers 4 and 5 (GPUs 2)
- Layer 10 (GPU3) -> Layers 11 and 8 (GPUs 4)

Overall Latency: Increased by inter-GPU communication delays.
```

**Data:**

```json
{
    "input": ["hello", "world"],
    "output": [
        {"token": "hello", "layer_outputs": [{"gpu1": [0.2, 0.3], "gpu2": [0.4, 0.5]}]},
        {"token": "world", "layer_outputs": [{"gpu3": [0.6, 0.7], "gpu4": [0.8, 0.9]}]}
    ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph LR;
3. A[Input Token "hello"] --> B{Is token split?};
4. B -- Yes --> C[Compute Layer 0 & 3 on GPU 1];
5. C --> D[Send Layer 3 output to GPUs 2,4];
6. D --> E{Is communication complete?};
7. E -- No --> F[Receive Layer 4/5 from GPU 2];
8. F --> G[Continue processing Layers 6-9 on GPU 3 & 4];
9. G --> H[Output Token "hello"];
10. H --> I[Output Token "world"];
11. I --> J{Is token split?};
12. J -- Yes --> K[Compute Layer 0,1 & 2 on GPUs 1,2];
13. K --> L[Send outputs to GPUs handling Layers 6-9];
14. L --> M[Receive results from GPUs 3,4 for Layers 6-9];
15. M --> N[Output Token "world"];
16. N --> O[End];

