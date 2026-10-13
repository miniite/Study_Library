---
type: "practice"
difficulty: "Intermediate"
---

# Which scikit-learn tool would you use to plot validation curves for a hyperparameter?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Ensembles and Variance Reduction](<../../../../11_AI_and_ML/ML_Foundations/Bias_Variance/Ensembles_and_Variance_Reduction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to understand that bias-variance tradeoff is crucial in machine learning, especially when tuning hyperparameters. Bias refers to the error due to overly simplistic models, while variance refers to the model's sensitivity to small fluctuations in the training data. Validation curves help us visualize how these errors change as a function of a specific hyperparameter.

The scikit-learn tool we would use for plotting validation curves is validation_curve. This function helps us analyze the effect of varying a hyperparameter on both the training and cross-validation scores, providing insights into where the model might be underfitting (high bias) or overfitting (high variance).

Let's break down how to use validation_curve step by step:

Import the Required Modules: We need to import validation_curve from sklearn.model_selection.
Prepare Your Model and Data: You should have a model, dataset, and the hyperparameter you want to tune.
Define the Hyperparameter Ranges: Specify the range of values for the hyperparameter you are interested in.
Compute Training and Validation Scores: validation_curve will compute these scores as it varies the specified hyperparameter.

Here’s a step-by-step walkthrough using a simple example:

Step 1: Import Required Libraries
PYTHON
from sklearn.model_selection import validation_curve
from sklearn.datasets import load_iris
from sklearn.svm import SVC
import matplotlib.pyplot as plt
Step 2: Prepare the Model and Data

For this example, let's use a Support Vector Machine (SVM) with an RBF kernel on the Iris dataset.

PYTHON
X, y = load_iris(return_X_y=True)
model = SVC(kernel='rbf')
param_range = [0.01, 0.1, 1, 10, 100]
Step 3: Compute Training and Validation Scores

We will plot the training score against the validation score for different values of the C hyperparameter.

PYTHON
train_scores, valid_scores = validation_curve(model, X, y, param_name="C", param_range=param_range, cv=5)
Step 4: Plot the Validation Curve

Now we can plot these scores to visualize the tradeoff between bias and variance.

PYTHON
plt.figure()
plt.plot(param_range, np.mean(train_scores, axis=1), label='Training score')
plt.plot(param_range, np.mean(valid_scores, axis=1), label='Cross-validation score')
plt.xlabel('C (hyperparameter)')
plt.ylabel('Score')
plt.legend(loc="best")
plt.show()

#### Closing Summary

Using validation_curve in scikit-learn allows us to visualize the impact of varying a hyperparameter on model performance. This helps identify the optimal range for that hyperparameter, balancing bias and variance.

#### Common Mistakes

Not Importing the Correct Function: Often, new users might confuse this with other validation functions.
Incorrect Hyperparameter Range: Choosing too few or too many values can lead to misleading results.
Ignoring Cross-Validation Scores: Focusing only on training scores without considering cross-validation can give a false sense of model performance.

