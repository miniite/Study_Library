---
type: "practice"
difficulty: "Basic"
---

# What is the difference between top-1 and top-5 accuracy in image classification benchmarks?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Classification Metrics](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Classification_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Top-1 and top-5 accuracy are common metrics used to evaluate the performance of models in image classification tasks. Top-1 accuracy measures how often a model correctly predicts the most likely class (the "top" prediction) among all possible classes, while top-5 accuracy considers both the correct class and the four highest-ranked classes.

Top-1 vs. Top-5 Accuracy: A Quick Overview

To understand the differences between these metrics, let’s start with some foundational concepts. In image classification, a model is trained to predict one of several possible classes for an input image. The task is to determine which class corresponds most accurately to the input.

#### Triage
and Assumptions

The question aims to clarify what accuracy metrics are used in machine learning models, specifically in computer vision tasks. We will assume that you have a basic understanding of how neural networks work in image classification.

#### First-Principles Solution
Top-1 Accuracy

Top-1 accuracy is the simplest and most straightforward metric. It calculates the percentage of times the model’s highest predicted probability class matches the ground truth label. In other words, for each input image, we check if the model predicts the correct class (the one with the highest confidence).

Top-5 Accuracy

Top-5 accuracy, on the other hand, is a more lenient metric. It considers that even if the model doesn’t predict the exact correct class as its top prediction, it still might be among the top five predicted classes. Therefore, for each input image, we check if the ground truth label appears in the top five predictions.

#### Example
Calculation

Assume an image has 10 possible classes: A, B, C, D, E, F, G, H, I, J.

If the model predicts [C, B, A, D, E] and the correct class is A, then:
Top-1 Accuracy: The prediction is incorrect because A (the ground truth) is not in the top one.
Top-5 Accuracy: Since A is in the first five predictions ([C, B, A, D, E]), the model gets this right.
Why Use Both Metrics?

Using both metrics provides a more comprehensive view of the model’s performance:

Top-1 accuracy gives an idea of how often the model makes confident and correct predictions.
Top-5 accuracy accounts for cases where the model is still reasonably close to being correct, even if it doesn’t pick the exact top prediction.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while top-1 accuracy focuses on the most confident prediction, top-5 accuracy considers a broader range of predictions. This means that models with lower top-1 accuracy might still perform well in terms of overall correctness if they frequently predict among the top five.

#### Common mistakes
include:

Overlooking the importance of both metrics and focusing only on one.
Not understanding how to calculate these metrics correctly, leading to incorrect reporting or interpretation.

#### Follow-ups
could involve discussing trade-offs between these metrics based on specific use cases, such as in medical imaging where misclassifications have severe consequences. Additionally, you might explore how different loss functions (like cross-entropy) can influence these accuracy metrics.

**Comparison:**

```
True Label	Predicted Labels (Top-1)	Predicted Labels (Top-5)
0	[2]	[2, 1]
2	[2]	[2, 3, 4, 1, 0]
3	[4]	[4, 2, 0, 1, 5]
4	[1]	[1, 2, 3, 0, 4]
Sample	Top-1 Prediction	Correct?	Top-5 Prediction	Correct?
Image 1	2	No	2, 1	Yes
Image 2	2	Yes	2, 3, 4, 1, 0	Yes
Image 3	4	No	4, 2, 0, 1, 5	No
Image 4	1	No	1, 2, 3, 0, 4	No
```

**Code:**

```python
# Example code to calculate top-1 and top-5 accuracy
import numpy as np

def calculate_accuracy(y_true, y_pred):
    correct_top_1 = 0
    correct_top_5 = 0

    for true_label, pred_labels in zip(y_true, y_pred):
        # Predicted probabilities
        top_1_index = np.argmax(pred_labels)
        if top_1_index == true_label:
            correct_top_1 += 1

        # Top-5 predictions
        top_5_indices = np.argsort(pred_labels)[-5:]
        if true_label in top_5_indices:
            correct_top_5 += 1

    total_samples = len(y_true)

    top_1_accuracy = correct_top_1 / total_samples
    top_5_accuracy = correct_top_5 / total_samples

    return top_1_accuracy, top_5_accuracy

# Example data
y_true = [0, 2, 3, 4]  # True labels
y_pred = [
    [0.1, 0.8, 0.1, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05],
    [0.05, 0.05, 0.8, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.1],
    [0.05, 0.05, 0.05, 0.85, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05],
    [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
]

top_1_acc, top_5_acc = calculate_accuracy(y_true, y_pred)
print(f"Top-1 Accuracy: {top_1_acc}")
print(f"Top-5 Accuracy: {top_5_acc}")
```

**Example input / output:**

```text
# Example input and output
y_true = [0, 2, 3, 4]
y_pred = [
    [0.1, 0.8, 0.1, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05],
    [0.05, 0.05, 0.8, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.1],
    [0.05, 0.05, 0.05, 0.85, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05],
    [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
]

top_1_acc, top_5_acc = calculate_accuracy(y_true, y_pred)
print(f"Top-1 Accuracy: {top_1_acc}")  # Output: Top-1 Accuracy: 0.25
print(f"Top-5 Accuracy: {top_5_acc}")  # Output: Top-5 Accuracy: 0.75
```

**Data:**

```json
{
  "y_true": [0, 2, 3, 4],
  "y_pred": [
    [0.1, 0.8, 0.1, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05],
    [0.05, 0.05, 0.8, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05, 0.1],
    [0.05, 0.05, 0.05, 0.85, 0.05, 0.05, 0.05, 0.05, 0.05, 0.05],
    [0.1, 0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0]
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is the model's top-1 predictioncorrect?
7. Top-1 Accuracy Increment
8. Is the ground truth in thetop-5 predictions?
9. Top-5 Accuracy Increment
10. No Top-1 or Top-5 Increment
11. End

