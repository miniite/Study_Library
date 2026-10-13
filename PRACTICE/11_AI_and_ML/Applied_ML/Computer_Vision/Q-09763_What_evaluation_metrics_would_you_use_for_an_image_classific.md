---
type: "practice"
difficulty: "Intermediate"
---

# What evaluation metrics would you use for an image classification problem, and how does accuracy fail on imbalanced classes?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Classification Metrics](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Classification_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating an image classification model in computer vision tasks, it's crucial to understand that different metrics are suited to different scenarios. For a balanced dataset, accuracy might suffice, but when dealing with imbalanced classes, accuracy can be misleading and other metrics become more important.

Firstly, let’s summarize the key points: we need evaluation metrics that provide a comprehensive understanding of model performance beyond just accuracy, especially in cases where data is imbalanced.

**Code:**

```python
# Example code to calculate some basic metrics using scikit-learn
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score

y_true = [0, 1, 0, 1, 0, 1, 1, 1]  # True labels
y_pred = [0, 0, 0, 1, 0, 1, 1, 1]  # Predicted labels

accuracy = accuracy_score(y_true, y_pred)
precision = precision_score(y_true, y_pred)
recall = recall_score(y_true, y_pred)
f1 = f1_score(y_true, y_pred)

print(f"Accuracy: {accuracy}")
print(f"Precision: {precision}")
print(f"Recall: {recall}")
print(f"F1 Score: {f1}")
```

