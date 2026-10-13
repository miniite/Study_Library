---
type: "practice"
difficulty: "Intermediate"
---

# How would you explain why pruning a decision tree reduces variance?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Ensembles and Variance Reduction](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Ensembles_and_Variance_Reduction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Pruning a decision tree is an effective technique to reduce model variance by simplifying the structure of the tree, which helps prevent overfitting. By removing branches that do not significantly contribute to predictive power, we balance between underfitting and overfitting.

#### Triage
The question asks you to explain why pruning reduces variance in a decision tree. Essentially, it's about understanding how reducing complexity can mitigate the risk of the model fitting too closely to the training data (overfitting). We need to assume that the dataset is noisy or complex enough to benefit from such simplification.

#### First-Principles Solution

Understanding Decision Trees and Variance: A decision tree's performance on unseen data largely depends on its complexity. High variance means the model may capture noise in the training data, leading to poor generalization.

Bias-Variance Decomposition Insight: The bias-variance tradeoff states that a model’s overall error is the sum of the bias and variance (plus irreducible error). For decision trees:

Bias represents the error from erroneous assumptions in the learning algorithm.
Variance captures how much the estimate of the target function will change if we train it on different subsets of the data.

Complexity and Overfitting: Unpruned decision trees can become too complex, leading to overfitting where they capture noise rather than the underlying pattern in the data. This is because the tree grows deeply to accommodate all possible splits, including those that are statistically insignificant.

Pruning Mechanism: Pruning involves cutting back some branches of the tree based on certain criteria such as reduced error or complexity (like cost complexity pruning). This process reduces the number of terminal nodes and thus the overall complexity of the model.

Impact on Variance Reduction: By reducing the depth and number of leaves, we decrease the likelihood that the tree will capture noise. Simpler models are less likely to overfit because they generalize better.

#### Closing Summary
+ Common Mistakes/Follow-Ups: In summary, pruning a decision tree reduces variance by simplifying the model structure, thereby making it more generalizable and robust against training data noise. A common mistake is underestimating the importance of complexity control; pruning must be done judiciously to avoid introducing bias. Follow-ups might include discussing different types of pruning techniques or how they interact with hyperparameter tuning.

**Code:**

```python
# Example of a simple decision tree without pruning
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier

data = load_iris()
X, y = data.data, data.target
tree = DecisionTreeClassifier(random_state=42)
tree.fit(X, y)

# Pruned version using cost complexity pruning ( ccp_alpha parameter )
pruned_tree = DecisionTreeClassifier(ccp_alpha=0.01, random_state=42)
pruned_tree.fit(X, y)
```

**Example input / output:**

| Input | Output (without pruning) | Output (with pruning) |
|-------|--------------------------|----------------------|
| X     | Predicted Class           | Predicted Class       |
| 5.1, 3.5, 1.4, 0.2   | 0                        | 0                    |
| 4.9, 3.0, 1.4, 0.2   | 0                        | 0                    |
| 7.0, 3.2, 4.7, 1.4   | 2                        | 2                    |

**Data:**

```json
{
  "data": {
    "X": [
      [5.1, 3.5, 1.4, 0.2],
      [4.9, 3.0, 1.4, 0.2],
      [7.0, 3.2, 4.7, 1.4]
    ],
    "y": [0, 0, 2]
  }
}
```

