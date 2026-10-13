---
type: "practice"
difficulty: "Intermediate"
---

# What is imbalanced data and why does it make preprocessing and evaluation harder?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Imbalanced Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Imbalanced_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with machine learning, we often encounter a situation where our dataset contains classes that are not evenly distributed—this is known as imbalanced data. For example, in a fraud detection system, fraudulent transactions might make up only 0.1% of all transactions. Such imbalance can significantly affect the model's performance and pose challenges during preprocessing and evaluation.

#### Triaging the Question

The core of this question asks us to understand why imbalanced data makes preprocessing and evaluation harder. The interviewer is probing our understanding of class distribution in datasets, its impact on models, and how we handle it effectively.

#### Assumptions
We assume a basic understanding of supervised learning and classification.
Imbalance refers specifically to skewed class distributions in the dataset.

#### First Principles Solution

Imbalanced data poses several challenges that can affect both model training and evaluation:

Model Bias: Models tend to be biased towards the majority class, which leads to poor performance on minority classes. For example, a classifier trained on an imbalanced dataset might predict the majority class almost all the time.
Evaluation Metrics: Traditional metrics like accuracy are misleading in imbalanced datasets because they can give a high score even if the model is not useful for the minority class. Precision, recall, F1-score, and AUC-ROC become more critical metrics to evaluate performance.
Class Imbalance Handling Techniques:
Resampling: Techniques like oversampling (increasing the number of samples in the minority class) or undersampling (decreasing the number of samples in the majority class).
Cost-Sensitive Learning: Assigning different misclassification costs to classes based on their importance.
Anomaly Detection: Treating imbalanced data as an anomaly detection problem.
Detailed Explanation

Let’s break down why these challenges arise and how we can address them:

Model Bias:

In a highly imbalanced dataset, if the model is not penalized for misclassifying minority class samples, it might prefer to predict the majority class almost always.

#### Example
If 99% of transactions are normal and only 1% are fraudulent, a simple model that always predicts “normal” would have an accuracy of 99%, which looks good on paper but is not useful in practice.

Evaluation Metrics:

Traditional metrics like Accuracy can be deceptive because they don't account for the true positive rate (recall) and precision.
For instance, if a model predicts every transaction as “normal,” it will have 99% accuracy, but recall would be 0%, which is not practical.

Resampling Techniques:

Oversampling: Adding more examples to the minority class can help improve learning about this class. However, this might lead to overfitting.
Undersampling: Reducing the majority class data can balance the dataset but may lose information.

Cost-Sensitive Learning:

Assigning higher cost to misclassifying the minority class ensures that the model pays more attention to these samples during training.

#### Example
In fraud detection, misclassifying a fraudulent transaction as normal could lead to significant financial losses.

Anomaly Detection:

Treating imbalanced data as an anomaly detection problem can be effective if we have clear boundaries between normal and abnormal behavior.

#### Closing Summary

In summary, dealing with imbalanced data requires careful consideration of both preprocessing techniques like resampling and choosing appropriate evaluation metrics. The key is to ensure that the model not only learns from the minority class but also performs well in real-world scenarios where misclassification costs can be high.

#### Common Mistakes/Follow-Ups
Mistake 1: Ignoring class imbalance without any correction, leading to biased models.

#### Follow-Up
1: Discuss how to choose between oversampling and undersampling techniques based on the specific dataset characteristics.
Mistake 2: Using only accuracy as an evaluation metric in imbalanced datasets.

#### Follow-Up
2: Explain other metrics like Precision, Recall, F1-Score, and AUC-ROC and when each is appropriate.

**Comparison:**

| Metric | Description                |
|--------|----------------------------|
| Accuracy | Proportion of correct predictions overall. Can be misleading in imbalanced datasets. |
| Precision | Proportion of true positives among predicted positives. Helps avoid false positives. |
| Recall (Sensitivity) | Proportion of actual positives that are correctly identified. |
| F1-Score | Harmonic mean of precision and recall, balanced between the two. |

**Code:**

```python
from sklearn.utils import resample

# Example of oversampling the minority class
minority_class = df[df['target'] == 'minority']
majority_class = df[df['target'] == 'majority']

# Upsample minority class
minority_class_upsampled = resample(minority_class, replace=True, n_samples=len(majority_class), random_state=123)
# Combine majority class with upsampled minority class
df_upsampled = pd.concat([majority_class, minority_class_upsampled])
```

**Example input / output:**

```text
Original dataset:
- normal (99%)
- fraudulent (1%)

Upsampled dataset:
- normal (50%)
- fraudulent (50%)
```

**Data:**

```json
{
  "data": {
    "target": ["normal", "normal", "normal", "fraudulent", "fraudulent"]
  }
}
```

**Flow:**

1. Identify Imbalance
2. Analyze Class Distribution
3. Choose Appropriate Metrics(Precision, Recall, F1-Score)
4. Consider ResamplingTechniques
5. Apply Cost-Sensitive Learning
6. Evaluate Model Performance
7. Implement Oversampling orUndersampling
8. Adjust Misclassification Costs

