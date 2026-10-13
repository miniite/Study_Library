---
type: "practice"
difficulty: "Deep"
---

# Explain how you would treat non-determinism (variable outputs at temperature > 0) as a first-class quality attribute in the utility tree, and give a scenario whose failure reveals the architecture has no design for bounding output variance across identical requests.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Regulatory Requirements in Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Regulatory_Requirements_in_Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address non-determinism in an AI system from a principled architectural perspective, we must treat it as a first-class quality attribute and integrate it into our utility tree. Non-determinism can introduce variability in model outputs, which is particularly problematic when performance consistency and predictability are critical. We need to design mechanisms that bound output variance across identical requests. This involves understanding the sources of non-determinism, quantifying its impact, and implementing compensatory strategies.

Let's start with a scenario where non-determinism causes significant issues: an AI-driven financial trading system. In this context, high-frequency trading (HFT) systems must operate with minimal latency and consistent performance to avoid missed trades or market inefficiencies. Suppose the trading model uses a large transformer-based neural network for predicting stock prices at temperature > 0. This introduces non-determinism due to stochasticity in sampling during inference.

**Comparison:**

```
Request ID	Predicted Price	Execution Time (s)	Order Timestamp
1	180.34	0.05	2023-10-01T14:00:00Z
2	180.56	0.07	2023-10-01T14:00:01Z
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Input Request] --> B[Model Inference (Stochastic Sampling)];
    B --> C[Non-Deterministic Output];
    C --> D[Variable Trade Execution Times];
    D --> E[Inconsistent Trade Orders Across Identical Requests];
```

**Example input / output:**

```

Given two identical requests for stock price prediction at the same time frame and stock symbol, we observe different execution times and predicted prices:

JSON
Copy
Request #1 -> Predicted Price: 180.34, Execution Time: 0.05s
Request #2 -> Predicted Price: 180.56, Execution Time: 0.07s

These variations in execution times and prices can lead to inconsistent trade orders across identical requests, impacting the overall performance and reliability of the trading system.
```

**Data:**

```json
{
  "request": {
    "stockSymbol": "AAPL",
    "timeFrame": "1m"
  },
  "outputVariation": [
    {
      "executionTime": "0.05s",
      "predictedPrice": 180.34,
      "orderTimestamp": "2023-10-01T14:00:00Z"
    },
    {
      "executionTime": "0.07s",
      "predictedPrice": 180.56,
      "orderTimestamp": "2023-10-01T14:00:01Z"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Input Request
4. Model Inference (StochasticSampling)
5. Is Temperature > 0?
6. Apply DeterministicConstraints
7. Predicted Price
8. Execute Trade Order
9. Variable Trade Execution Times
10. Consistent Trade Orders AcrossIdentical Requests
11. Inconsistent Trade OrdersAcross Identical Requests

