---
type: "practice"
difficulty: "Basic"
---

# What is an outlier and how can it affect the training of a machine learning model?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Imbalanced Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Imbalanced_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

Outliers are data points that significantly differ from other observations in a dataset, often making them seem unusual or out-of-place. These anomalies can profoundly impact the performance and reliability of machine learning models during training by skewing statistical measures such as mean and standard deviation, leading to suboptimal model parameters and potentially misleading insights.

#### Triaging the Question

The question is asking you to explain what outliers are and their potential impacts on a machine learning model's training process. The interviewer might be probing your understanding of basic statistical concepts and how these affect data analysis and modeling. Assumptions include familiarity with common types of anomalies in datasets, and recognizing scenarios where they could arise.

#### First-Principles Solution
Definition: An outlier is a data point that deviates significantly from other observations within the same dataset.
Impact on Training:
Statistical Measures: Outliers can distort mean, median, and standard deviation calculations, leading to biased model parameters.
Model Performance: They might cause models to fit poorly or overly focus on unusual patterns, impacting prediction accuracy.
Visualization: Visualizations like box plots, scatter plots, etc., can highlight outliers easily.

To mitigate these effects: 3. Detection Techniques:

Z-Score and IQR (Interquartile Range) methods are commonly used to identify outliers.
Handling Outliers:
Removal: If the outliers are due to data entry errors or measurement mistakes, they can be removed.
Transformation: Applying transformations like winsorization (clipping extreme values).
Imputation: Filling in missing data with appropriate methods.

#### Closing Summary

Understanding and managing outliers is crucial for building robust machine learning models. Proper handling ensures that statistical measures are reliable and model performance is not compromised by anomalies.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the presence of outliers can lead to erroneous conclusions.

#### Follow-Up
What steps would you take if your dataset contains a large number of outliers? How does this differ from scenarios with fewer or no outliers?

**Code:**

```python
# Example code for detecting and handling outliers using IQR method
import numpy as np

def detect_outliers_IQR(data):
    Q1 = np.percentile(data, 25)
    Q3 = np.percentile(data, 75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    return (data < lower_bound) | (data > upper_bound)

# Example usage
outliers = detect_outliers_IQR([2, 4, 6, 8, 100])
print(outliers)
```

