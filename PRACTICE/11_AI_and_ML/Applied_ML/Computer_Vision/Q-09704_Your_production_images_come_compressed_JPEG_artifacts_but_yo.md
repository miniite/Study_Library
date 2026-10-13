---
type: "practice"
difficulty: "Deep"
---

# Your production images come compressed (JPEG artifacts) but you trained on clean images. How would this mismatch affect the model and how would you fix it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data vs Model Improvements](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Data_vs_Model_Improvements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with a machine learning model that was trained on clean images, but encounters production data with JPEG artifacts (compression artifacts), there will be a mismatch between the training and inference environments. This can lead to reduced performance or unexpected behavior in the model’s predictions. To understand and mitigate this issue, we need to explore how these artifacts affect the model and what steps we can take to address them.

#### Triaging the Question

The question asks us to consider how JPEG compression artifacts impact a machine learning model trained on clean images. The core of the problem is that while the training data does not contain any artifacts, the production environment introduces distortions due to compression. This mismatch could lead to degraded performance or even incorrect predictions.

#### Assumptions
and Scope
Model Type: We assume it’s a Convolutional Neural Network (CNN) trained for some computer vision task.
Data Distribution: Training data is assumed to be clean, while production data contains JPEG artifacts.
Impact on Performance: The focus is on understanding the effects on model performance and proposing solutions.
Step-by-Step Solution

Understand Artifacts:

JPEG Artifacts: Compression introduces distortions like blocky patterns, color blurring, or aliasing in the images.
Effect on Model: These artifacts can make it harder for the model to recognize features correctly, leading to reduced accuracy.

Model Evaluation:

Initial Baseline: Run predictions on a sample of production data with and without JPEG artifacts.
Performance Metrics: Compare metrics like accuracy, F1 score, precision, recall, etc., between clean and compressed images.

Feature Analysis:

Feature Importance: Analyze which features are most affected by compression using techniques like SHAP values or LIME (Local Interpretable Model-agnostic Explanations).
Filtering Artifacts: Identify specific artifacts that significantly impact the model’s performance.

Model Retraining:

Data Augmentation: Introduce data augmentation to simulate JPEG artifacts in training.
PYTHON
from albumentations import CompressImage, ImageOnlyTransform

augmentations = [
    CompressImage(quality_lower=70, quality_upper=95),
]
Fine-Tuning: Retrain the model on this augmented dataset to better handle production data.

Model Architecture:

Custom Loss Functions: Modify loss functions to penalize errors introduced by artifacts.
Regularization Techniques: Introduce techniques like dropout or weight decay to improve generalization.

Deployment Strategy:

Threshold Adjustment: Adjust prediction thresholds based on the nature of the artifacts and their impact.
Real-Time Compression Detection: Implement a real-time detection system to flag images with excessive artifacts before model inference.

#### Closing Summary

JPEG artifacts significantly affect model performance by introducing distortions that are not present in the training data. To mitigate this, we can:

Evaluate the model’s performance on compressed data.
Identify and filter out highly impactful artifacts.
Augment the training data with JPEG artifacts.
Retrain the model using these augmented datasets.
Implement real-time compression detection to improve deployment robustness.

#### Common Mistakes/Follow-Ups
Ignoring Artifact Impact: Failing to acknowledge how JPEG artifacts degrade performance.
Overfitting on Clean Data: Relying too heavily on clean training data without considering the impact of production artifacts.
Inadequate Retraining: Not retraining the model with sufficient data augmentation or fine-tuning techniques.

**Code:**

```python
from albumentations import CompressImage, ImageOnlyTransform

augmentations = [
    CompressImage(quality_lower=70, quality_upper=95),
]
```

