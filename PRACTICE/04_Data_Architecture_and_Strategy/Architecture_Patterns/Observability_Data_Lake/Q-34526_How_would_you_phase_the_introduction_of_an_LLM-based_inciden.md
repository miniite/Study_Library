---
type: "practice"
difficulty: "Expert"
---

# How would you phase the introduction of an LLM-based incident copilot on top of the lake so it earns trust before it's allowed to trigger any automated action?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Mesh vs Central Observability Lake](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Mesh_vs_Central_Observability_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To introduce an LLM-based incident copilot on top of a Observability Data Lake, we need to ensure that the system is reliable and trusted by the operations team. The key steps include phased deployment, continuous monitoring, iterative refinement, and establishing clear guidelines for when automated actions are permissible.

The core idea here is to incrementally build trust in the LLM model through controlled experiments and feedback loops before integrating it into high-stakes decision-making processes. This involves starting with a small-scale pilot, collecting data on its performance, and iteratively improving both the model and the governance mechanisms until we can safely allow it to trigger automated actions.

**Comparison:**

```
Timestamp	Accuracy	False Positives	False Negatives
2023-10-01	0.95	3	2
2023-10-08	0.97	1	1
```

**Code:**

```python
# Example of monitoring code for LLM-based copilot
import pandas as pd

def monitor_copilot_performance(predictions_df):
    # Calculate key metrics such as accuracy, false positives/negatives
    accuracy = (predictions_df['predicted'] == predictions_df['actual']).mean()
    false_positives = len(predictions_df[predictions_df['predicted'] & ~predictions_df['actual']])
    false_negatives = len(predictions_df[~predictions_df['predicted'] & predictions_df['actual']])

    return {
        'accuracy': accuracy,
        'false_positives': false_positives,
        'false_negatives': false_negatives
    }
```

**Example input / output:**

```text
Input:
{
  "predictions_df": pd.DataFrame({
      'timestamp': ['2023-10-01', '2023-10-08'],
      'predicted': [True, True],
      'actual': [False, True]
    })
}

Output:
{
  'accuracy': 0.95,
  'false_positives': 3,
  'false_negatives': 2
}
```

**Data:**

```json
{
  "data": [
    {"timestamp": "2023-10-01", "accuracy": 0.95, "false_positives": 3, "false_negatives": 2},
    {"timestamp": "2023-10-08", "accuracy": 0.97, "false_positives": 1, "false_negatives": 1}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the LLM model ready?
5. Maintain manual monitoring
6. Review and refine model
7. Phase in copilot withautomated actions
8. Repeat if necessary
9. Deploy fully, allow automatedactions

