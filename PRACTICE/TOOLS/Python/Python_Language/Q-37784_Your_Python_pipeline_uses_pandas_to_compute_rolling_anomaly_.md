---
type: "practice"
difficulty: "Deep"
---

# Your Python pipeline uses pandas to compute rolling anomaly stats but chokes on 50k concurrent series. How would you decide between vectorized numpy, incremental online updates, or moving to a streaming engine?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Data Migrations in Python](<../../../../TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of processing 50k concurrent time series in Python using pandas and rolling anomaly stats, we need to consider the bottlenecks and trade-offs between vectorized operations with numpy, incremental online updates, and moving to a streaming engine. This involves understanding the nature of our data and the performance characteristics of each approach.

The core question here is whether we can optimize the current implementation or if we need to reconsider our architecture. We'll start by triaging the problem and then evaluate each option from first principles.

**Comparison:**

| **Approach** | **Pros** | **Cons** |
| --- | --- | --- |
| Vectorized numpy | Fast, optimized | Limited to specific use cases, harder to implement |
| Incremental online updates | Memory efficient, scalable | More complex implementation, potential for numerical instability |
| Streaming engine | Highly scalable, fault-tolerant | Complexity in setup and integration, may require additional infrastructure |

**Code:**

```python
import pandas as pd
import numpy as np

# Example data setup
data = pd.DataFrame({'timestamp': pd.date_range(start='2023-10-01', periods=500, freq='H'),
                     'value': np.random.randn(500)})
```

**Example input / output:**

```python
# Initial pandas rolling window computation
rolling_stats = data.rolling(window=10).mean()
print(rolling_stats.head())

To understand the problem, let's first consider why pandas is choking on 50k concurrent series. Pandas operations are primarily designed for in-memory computation and can struggle with large datasets due to memory constraints and computational overhead.
```

**Data:**

```json
{
  "data": {
    "timestamp": ["2023-10-01 00:00:00", "2023-10-01 01:00:00", ..., "2023-10-04 23:00:00"],
    "value": [0.5, -0.7, ..., 0.8]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Initial pandas processing
6. Is the dataset small enough tofit into memory?
7. Use vectorized numpy
8. Can we implement incrementalupdates?
9. Implement incrementalupdates and optimize
10. Migrate to a streaming engine
11. Optimize memory usage andperformance
12. Test the implementationthoroughly
13. Set up and integrate streamingengine

