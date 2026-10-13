---
type: "practice"
difficulty: "Intermediate"
---

# The AI writes code that computes R-squared for a regression. What edge case (constant predictions, tiny sample) would you test?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Verifying AI Generated ML Code](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Verifying_AI_Generated_ML_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure robustness in an AI-assisted coding system like Copilot or similar tools, it's crucial to test edge cases that can reveal potential issues with the R-squared computation. The key edge case we need to consider is when the predictions are constant across all data points. This scenario poses a significant challenge because R-squared, which measures how well future outcomes are likely to be predicted by the model, might not behave as expected.

Let's break down why this edge case is important and walk through the solution step-by-step.

**Comparison:**

| True Values | Predicted Values |
|-------------|------------------|
|    3.5      |       3.5        |
|    3.6      |       3.5        |
|    3.4      |       3.5        |
|    3.7      |       3.5        |

**Code:**

```python
# Sample code for testing R-squared computation with constant predictions
import numpy as np

def compute_r_squared(y_true, y_pred):
    """Compute the R-squared value given true and predicted values."""
    ss_res = np.sum((y_true - y_pred) ** 2)
    ss_tot = np.sum((y_true - np.mean(y_true)) ** 2)
    r_squared = 1 - (ss_res / ss_tot)
    return r_squared

# Test data with constant predictions
y_true = [3.5, 3.6, 3.4, 3.7]
y_pred = [3.5] * len(y_true)  # Constant prediction of 3.5
r_squared = compute_r_squared(y_true, y_pred)
print(f"R-squared: {r_squared}")
```

**Example input / output:**

```text
Input:
- True values: y_true = [3.5, 3.6, 3.4, 3.7]
- Predicted values: y_pred = [3.5, 3.5, 3.5, 3.5]

Output:
R-squared: 1.0
```

**Data:**

```json
{
    "y_true": [3.5, 3.6, 3.4, 3.7],
    "y_pred": [3.5, 3.5, 3.5, 3.5]
}
```

**Flow:**

1. Yes
2. No
3. Input Data
4. Is the prediction constant?
5. R-squared = 1.0
6. Calculate R-squared normally
7. Output R-squared
8. Compute SS_res and SS_tot,then calculate R-squared

