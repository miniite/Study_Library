---
type: "practice"
difficulty: "Deep"
---

# Your evaluation and safety team wants shadow-scoring of every production response with a second model for drift and harm detection, doubling per-request cost. How do you govern the sampling rate of shadow eval so you catch regressions without paying to score every single request?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cost Anomaly Detection](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Cost_Anomaly_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of doubling the per-request cost through shadow evaluation while ensuring that we catch regressions, we need a strategic approach to sampling. The core idea is to balance between catching as many regressions as possible and minimizing unnecessary costs. We will outline the key steps to achieve this.

Understand the Problem: Doubling the cost by scoring every request with an additional model introduces significant overhead. Our goal is to sample requests in such a way that we can identify critical issues without overly burdening the system.
Define Sampling Criteria: We need a method to select which requests should be shadow-evaluated. This involves understanding what constitutes a "critical" response and how often these responses occur.
Risk-Based Sampling: Implement a risk-based sampling strategy where more frequent or high-risk scenarios are prioritized for evaluation.

The key concept here is to use statistical methods and domain expertise to guide the sampling rate, ensuring that we capture regressions effectively without incurring excessive costs.

**Comparison:**

| ID | Timestamp          | Response |
|----|--------------------|----------|
| 1  | 2023-10-01T09:00:00Z| success  |
| 2  | 2023-10-01T09:05:00Z| failure  |
| 3  | 2023-10-01T09:10:00Z| success  |
| 4  | 2023-10-01T09:15:00Z| failure  |

**Code:**

```python
# Example of a basic sampling function using random selection
import random

def sample_requests(requests, sampling_rate):
    sampled_requests = []
    for i, request in enumerate(requests):
        if random.random() < sampling_rate:
            sampled_requests.append(request)
    return sampled_requests
```

**Example input / output:**

```text
Input:
Sampling rate = 0.5 (50%)

Sampled requests using the sampling function:
[{"id": 1, "timestamp": "2023-10-01T09:00:00Z", "response": "success"},
 {"id": 4, "timestamp": "2023-10-01T09:15:00Z", "response": "failure"}]
```

**Data:**

```json
{
  "requests": [
    {"id": 1, "timestamp": "2023-10-01T09:00:00Z", "response": "success"},
    {"id": 2, "timestamp": "2023-10-01T09:05:00Z", "response": "failure"},
    {"id": 3, "timestamp": "2023-10-01T09:10:00Z", "response": "success"},
    {"id": 4, "timestamp": "2023-10-01T09:15:00Z", "response": "failure"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Request
4. Sample Request?
5. Shadow Evaluated
6. Processed as Normal
7. Analyze Results forRegressions
8. Alerts on Critical Issues

