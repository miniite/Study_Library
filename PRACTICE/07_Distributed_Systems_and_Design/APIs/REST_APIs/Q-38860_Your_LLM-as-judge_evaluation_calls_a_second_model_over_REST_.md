---
type: "practice"
difficulty: "Deep"
---

# Your LLM-as-judge evaluation calls a second model over REST for every candidate output. Under load the judge calls dominate cost and latency. How would you batch, cache, and rate-shape the judge REST calls without skewing scores?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Load Shedding Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Load_Shedding_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating candidate outputs from an LLM using a second model over REST, the primary challenge is managing the cost and latency under load while ensuring that the evaluation results remain accurate and unbiased. To address this, we need to implement a robust batching, caching, and rate-shaping strategy for these REST calls.

**Comparison:**

```
Batch Size	Cache Hit Rate	Evaluation Accuracy
Small	Low	High
Medium	Medium	Medium
Large	High	Low
```

**Code:**

```python
# Example Python snippet for rate shaping using `ratelimiter` package
from ratelimiter import RateLimiter

def evaluate_candidate_output(candidate_output):
    with RateLimiter(max_calls=10, period=1):  # Limit to 10 calls per second
        response = requests.post('https://second-model-api.com/evaluate', json={"input": candidate_output})
        return response.json()
```

**Example input / output:**

```

Given the input:

JSON
Copy
[
    "This is a test output.",
    "Another test output with different content."
]

The output would be:

JSON
Copy
[
    {"evaluation_score": 0.85, "confidence_intervals": [0.79, 0.92]},
    {"evaluation_score": 0.63, "confidence_intervals": [0.58, 0.68]}
]
```

**Data:**

```json
{
    "candidate_outputs": [
        "This is a test output.",
        "Another test output with different content."
    ]
}
```

**Flow:**

1. Small
2. Medium
3. Large
4. Valid
5. Invalid
6. Input Candidate Outputs
7. Batch Size?
8. Cache Miss
9. Cached Output
10. Rate-Limited Call
11. Evaluate with Second Model
12. Return Cached Evaluation
13. Evaluation Response?
14. Process & Cache Result

