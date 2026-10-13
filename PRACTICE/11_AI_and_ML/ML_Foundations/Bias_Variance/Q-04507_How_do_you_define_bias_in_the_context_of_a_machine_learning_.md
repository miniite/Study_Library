---
type: "practice"
difficulty: "Basic"
---

# How do you define bias in the context of a machine learning model?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Diagnosing Bias and Variance](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Diagnosing_Bias_and_Variance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In machine learning, bias refers to the error introduced by approximating a real-world problem with a simplified model. It is essentially the difference between the expected (or average) prediction of our model and the correct value which we are trying to predict. In simpler terms, it captures the systematic deviation from the true values caused by overly simplistic assumptions in the learning algorithm.

#### Triage

The question is asking for a definition of bias specifically within the context of machine learning models. The interviewer might be probing your understanding of how bias impacts model performance and how it can be managed or mitigated.

#### First-Principles Solution

To understand bias, we need to start with the core idea that our goal in training a machine learning model is to make predictions as close as possible to the true values. Bias arises when the model’s assumptions cause it to systematically learn incorrect patterns from the data, leading to systematic errors.

Defining Bias:
Simplified Model Assumptions: In many cases, we use simple models because they are easier to understand and compute. These models make strong assumptions about the underlying relationships in the data. If these assumptions do not hold true for the real-world problem, our model will consistently make errors.
Underfitting: When a model is too simple to capture the complexity of the underlying data distribution, it underfits the training data and produces predictions that are systematically incorrect. This results in high bias.
How Bias Affects Model Performance:
High Bias (Underfitting): The model is too simplistic to learn from the data effectively. It makes large errors both during training and testing.
Low Bias: The model has a more flexible structure, which can capture the underlying patterns in the data. However, if it's overly complex, it might start learning noise or outliers in the data.

#### Closing Summary

In summary, bias in machine learning is about making strong simplifying assumptions that may not accurately reflect the true underlying relationships in the data. High bias leads to underfitting and systematic errors. Understanding and managing bias is crucial for building effective models.

#### Common Mistakes/Follow-Ups
Mistake: Confusing bias with variance. While both are types of error, they measure different things—bias measures the consistency (systematic errors) while variance measures how sensitive the model's predictions are to small fluctuations in the training data.

#### Follow-Up
The interviewer might ask about techniques to reduce bias or balance it against variance.

**Comparison:**

```
True Values	Predicted Values	Difference
1.0	0.5	-0.5
2.0	1.5	-0.5
3.0	2.5	-0.5
```

**Code:**

```text
# Example code for calculating model bias
def calculate_bias(true_values, predicted_values):
    return np.mean(true_values - predicted_values)
```

**Example input / output:**

```
Example input and output for the bias calculation function

true_values = np.array([1.0, 2.0, 3.0]) predicted_values = np.array([0.5, 1.5, 2.5])

bias = calculate_bias(true_values, predicted_values) print(f"Bias: {bias}")

CODE
Copy
Output:

Bias: -0.5
```

**Data:**

```json
{
  "true_values": [1.0, 2.0, 3.0],
  "predicted_values": [0.5, 1.5, 2.5]
}
```

**Flow:**

1. No
2. True Values
3. Predicted Values
4. Are they the same?
5. Bias = True - Predicted

