---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd approach a semi-supervised setup where you have 5k labeled and 500k unlabeled images to boost a classifier.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data vs Model Improvements](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Data_vs_Model_Improvements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a semi-supervised setting with 5,000 labeled and 500,000 unlabeled images, the goal is to leverage both types of data to improve the performance of a classifier while managing computational constraints. This approach involves using self-training or pseudo-labeling techniques to tag some of the unlabeled data based on predictions from the current model. Here’s how I would tackle this problem:

#### Triage
and Core Idea

The key idea is to balance between leveraging the abundant unlabeled data for training a better classifier while ensuring that the few labeled examples remain high-quality inputs. The goal is to iteratively improve the model by incorporating reliable pseudo-labels from the unlabeled set, which can then be fed back into the training process.

#### First-Principles Solution
Initial Training: Start with the 5,000 labeled images and train a base classifier.
Pseudo Labeling: Use the trained model to predict labels for the 500,000 unlabeled images. These predictions serve as pseudo-labels.
Filtering Pseudo-Labels: Filter out low-confidence or incorrect pseudo-labels by comparing them against a validation set if available, or by applying domain-specific knowledge.
Leverage Active Learning: Select the most confident and informative unlabeled examples to be manually annotated for future training iterations (if resources allow).
Iterative Training: Incorporate both labeled and filtered pseudo-labeled data into the training process, potentially using techniques like mixup or uncertainty weighting to balance contributions from different sources.
Validation and Testing: Continuously validate the model on a held-out test set and refine as necessary.
Detailed Steps

Initial Model Training:

Use a Convolutional Neural Network (CNN) architecture suitable for computer vision tasks, such as ResNet or EfficientNet.
Train this initial model using the 5,000 labeled images only to establish a baseline performance.

Pseudo Labeling:

Run the trained model on all 500,000 unlabeled images and assign pseudo-labels based on the highest confidence prediction for each image.
Evaluate these pseudo-labeled predictions against ground truth (if available) or use domain knowledge to filter out low-confidence labels.

Filtering Pseudo-Labels:

Implement a threshold-based approach where only predictions with high confidence scores are considered valid.
Alternatively, use a cross-validation method where the model is retrained multiple times on subsets of labeled data and pseudo-labels, validating each iteration to filter out unreliable labels.

Leverage Active Learning:

Use an active learning strategy to select the most informative unlabeled examples for manual annotation. Techniques like uncertainty sampling can be particularly effective.
This step is crucial if additional annotations are a significant cost but can significantly improve model performance over time.

Iterative Training and Validation:

Combine both labeled data and filtered pseudo-labeled data in each training iteration, ensuring that the model weights are updated based on contributions from all sources.
Use strategies like uncertainty weighting or mixup to balance the impact of different types of inputs during training.

Continuous Monitoring and Optimization:

Regularly monitor the performance of the model on a validation set and adjust hyperparameters as needed.
Explore different architectures, pre-trained models, and regularization techniques to optimize for better generalization.

#### Closing Summary

By iteratively incorporating pseudo-labeled data into the training process, we can leverage the vast amount of unlabeled images to significantly boost classifier performance without overfitting or compromising on quality. This approach requires careful filtering of unreliable labels and strategic use of active learning to ensure that resources are used efficiently.

#### Common Mistakes
and Follow-Ups
Over-reliance on Pseudo-Labels: Failing to filter out low-confidence pseudo-labels can lead to poor model performance.
Ignoring Domain Knowledge: Relying solely on automated methods for filtering pseudo-labels without incorporating domain-specific insights can result in suboptimal outcomes.
Neglecting Active Learning: Over-relying on labeled data while ignoring the potential value of unlabeled images can limit the overall improvement.

**Comparison:**

```
Filtered Labels	Confidence
cat	0.95
dog	0.8
```

**Code:**

```python
# Example Pseudo Label Filtering Function
import numpy as np

def filter_pseudo_labels(predictions, ground_truth=None, confidence_threshold=0.9):
    filtered_labels = []
    for pred in predictions:
        if pred['confidence'] > confidence_threshold:
            if ground_truth and pred['label'] == ground_truth[pred['image_id']]:
                filtered_labels.append(pred)
            else:
                # Additional checks using domain knowledge
                pass
    return [p['label'] for p in filtered_labels]
```

**Example input / output:**

```text
# Example Input and Output for Filtering Pseudo-Labels

filtered_labels = filter_pseudo_labels(predictions, ground_truth)
print(filtered_labels) # Output: ['cat', 'dog']
```

**Data:**

```json
{
  "predictions": [
    {"image_id": 1, "label": "cat", "confidence": 0.95},
    {"image_id": 2, "label": "dog", "confidence": 0.8},
    {"image_id": 3, "label": "bird", "confidence": 0.7}
  ],
  "ground_truth": {
    "1": "cat",
    "2": "dog",
    "3": "bird"
  }
}
```

**Flow:**

1. Yes
2. No
3. Initial Training
4. Pseudo Labeling
5. Filter Low-Confidence Labels
6. Leverage Active Learning
7. Iterative Training
8. Monitor Performance

