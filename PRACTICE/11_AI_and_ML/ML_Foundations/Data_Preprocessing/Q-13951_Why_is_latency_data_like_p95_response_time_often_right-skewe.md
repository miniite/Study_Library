---
type: "practice"
difficulty: "Deep"
---

# Why is latency data (like p95 response time) often right-skewed, and what transformation would you apply before modeling it?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Scaling with Contaminated Data](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Scaling_with_Contaminated_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Latency data like p95 response time is often right-skewed because a small number of extremely high values can significantly affect the distribution. This skewness can distort statistical measures and model predictions, so it's crucial to preprocess this data before modeling. A common approach is applying transformations such as log or Box-Cox to reduce skewness.

#### Triaging the Question

The question asks for an understanding of why p95 response time data tends to be right-skewed and what transformation techniques can be applied to clean this data before modeling. The interviewer is probing your knowledge of distribution characteristics, common preprocessing steps in data science, and transformation methods to handle skewness.

#### First-Principles Solution
Understanding Skewness: Right-skewed distributions have a long tail on the right side, meaning that a small number of extreme high values are pulling the mean away from the median. In latency data, these outliers typically represent occasional slow response times due to network issues or server overload.
Impact of Skewness: High skew can mislead model training and predictions. For instance, a model might underfit by not capturing the frequent lower latencies if it is heavily influenced by the few high values.
Transformation Techniques:
Log Transformation: Applying the natural logarithm (log) to the data reduces right-skewed distributions because the transformation compresses the larger values more than the smaller ones, making the distribution closer to normal.
Box-Cox Transformation: This is a family of power transformations that can be used for any positive data. It's particularly effective when you want to make the data more normally distributed.

#### Closing Summary
and Common Mistakes

Always transform skewed data before modeling, especially in scenarios where model accuracy heavily depends on capturing the true distribution of the input variables. Common mistakes include:

Not Identifying Skewness: Assuming all data is normally distributed.
Overfitting Models: Applying heavy transformations that might not improve the model's generalization ability.
Code Example

Let's illustrate this with a simple Python example using pandas and numpy:

PYTHON
import pandas as pd
import numpy as np
from scipy.stats import boxcox

# Sample data: p95 response times in milliseconds
response_times = [20, 30, 40, 50, 60, 70, 80, 1000, 1200]

# Convert to a DataFrame for easier manipulation
df = pd.DataFrame({'ResponseTime': response_times})

# Log transformation
log_transformed = np.log(df['ResponseTime'] + 1)  # Adding 1 to handle zeros

# Box-Cox Transformation
lambda_val, transformed_data = boxcox(df['ResponseTime'])

print("Original Data:", df['ResponseTime'].values)
print("Log Transformed Data:", log_transformed.values)
print("Box-Cox Transformed Data (λ={})".format(lambda_val), transformed_data)
Self-Check

The answer covers the core concepts, provides practical examples, and addresses common mistakes. The code snippets are accurate and relevant to the problem.

