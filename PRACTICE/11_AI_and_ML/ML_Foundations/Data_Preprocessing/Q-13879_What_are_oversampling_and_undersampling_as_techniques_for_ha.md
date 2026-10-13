---
type: "practice"
difficulty: "Intermediate"
---

# What are oversampling and undersampling as techniques for handling class imbalance?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Imbalanced Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Imbalanced_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Oversampling and undersampling are two common data preprocessing techniques used in dealing with class imbalance in datasets, particularly in machine learning problems where one class significantly outnumbers another. Oversampling involves increasing the number of instances in the minority class either by generating synthetic samples or duplicating existing ones. Undersampling, on the other hand, reduces the number of majority class instances to balance the classes. Both techniques aim to create a more balanced dataset that can improve model performance and reduce bias.

Let's break down these concepts from first principles:

Understanding Class Imbalance: In many classification problems, one class (the minority) has far fewer samples than another class (the majority). This imbalance can lead to models being biased towards the majority class during training, thereby underperforming on the minority class.

Oversampling:

Techniques: Common oversampling techniques include SMOTE (Synthetic Minority Over-sampling Technique), ADASYN (Adaptive Synthetic Sampling), and Borderline-SMOTE.
SMOTE Example: Imagine we have a dataset where the positive class is much smaller than the negative class. Using SMOTE, we can generate new synthetic samples for the minority class by interpolating between existing ones.

Undersampling:

Techniques: Common undersampling techniques include random undersampling and cluster centroids.
Random Undersampling Example: If the majority class has many instances, you could randomly remove some of them to balance out with the minority class. Cluster centroids involves creating a new instance at the centroid of each cluster formed by the majority class.

#### Trade-offs

Oversampling can introduce noise and overfitting if too many synthetic samples are generated.
Undersampling risks losing important information from the majority class, which might contain valuable patterns that could be crucial for accurate predictions.

Choosing the Right Technique: The choice between oversampling and undersampling depends on the specific problem. For instance, oversampling is often used in medical diagnostics where missing a case of a rare condition can have severe consequences, while undersampling might be preferred in applications like spam filtering to avoid marking legitimate emails as spam.

Implementing Techniques:

In Python using imbalanced-learn library, you could use SMOTE for oversampling with the following code snippet:
PYTHON
from imblearn.over_sampling import SMOTE

# Assuming X is features and y are labels
sm = SMOTE(random_state=42)
X_resampled, y_resampled = sm.fit_resample(X, y)

#### Example
Consider a dataset where the minority class has 10 samples, and the majority class has 1000 samples. By applying SMOTE, you could generate synthetic samples to bring the minority class closer to balance with the majority.

**Code:**

```python
from imblearn.over_sampling import SMOTE

# Example usage of SMOTE
sm = SMOTE(random_state=42)
X_resampled, y_resampled = sm.fit_resample(X, y)
```

