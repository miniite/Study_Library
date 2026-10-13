---
type: "practice"
difficulty: "Intermediate"
---

# Why can label encoding be problematic for nominal categorical variables?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Missing Values and Encoding](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Missing_Values_and_Encoding.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Label encoding is a common technique to convert categorical data into numerical values, but it can lead to issues when applied to nominal categorical variables because these methods imply an ordinal relationship that doesn't exist.

#### Triage

The core of this question revolves around understanding why label encoding might mislead the model and distort its interpretation of nominal categories. The key idea is that for nominal categorical data, such as hair color or types of animals, there's no inherent order; however, using label encoding can make the model think these variables have a natural ranking.

First Principles
Understanding Nominal Categorical Variables: These are categories with no intrinsic order, like "cat," "dog," and "bird." They represent distinct and unrelated items.
Label Encoding: This involves assigning an integer to each category. For example:
PYTHON
hair_color = {"blonde": 0, "brown": 1, "red": 2}
Problematic Assumptions: By encoding as integers, the model might assume a numerical relationship (e.g., "brown" is more than "blonde," and "red" is the highest), which doesn't make sense for nominal data.
Step-by-Step Reasoning

Numerical Relationship Misinterpretation:

#### Cons
ider two hair colors: "blonde" and "brown." If label encoding assigns them 0 and 1, a model might interpret this as a numerical increase (e.g., "blonde < brown").
In reality, there's no such order. Both are just different categories.

Impact on Model Performance:

Models like decision trees or linear models may get incorrect feature importance scores.
This can lead to poor model performance because the model incorrectly interprets these categorical values as numerical.

#### Example
with Decision Trees:

PYTHON
from sklearn.datasets import load_iris
from sklearn.tree import DecisionTreeClassifier

iris = load_iris()
X, y = iris.data, iris.target  # Iris dataset has ordinal categories

model = DecisionTreeClassifier()
model.fit(X, y)

print(model.feature_importances_)
For the iris dataset (which has ordinal categories), this works fine. However, if you encode nominal data like hair color using label encoding and fit a decision tree:
PYTHON
X['hair_color'] = ['blonde', 'brown', 'red']
# Incorrectly treating them as integers 0, 1, 2

model.fit(X[['hair_color']], y)
The feature importance would be misinterpreted due to the numerical relationship assumption.

Addressing Nominal Data:

Use one-hot encoding instead of label encoding for nominal data.
PYTHON
from sklearn.preprocessing import OneHotEncoder

hair_color = pd.DataFrame({"hair_color": ["blonde", "brown", "red"]})
encoder = OneHotEncoder(sparse=False)
transformed = encoder.fit_transform(hair_color)

print(transformed)
This approach creates binary columns for each category, avoiding the ordinal assumption.

#### Closing Summary

Label encoding nominal categorical variables can lead to misleading interpretations of the data by implying a numerical relationship that does not exist. For such cases, one-hot encoding is preferred as it avoids these issues and correctly represents the categories without any implied order.

#### Common Mistakes
and Follow-Ups
Mistake: Using label encoding on nominal data.

#### Follow-Up
Discussing scenarios where ordinal vs. nominal categorical variables differ, like in churn prediction (ordinal) versus gender (nominal).
Further Reading: Exploring other encoding techniques like target encoding for specific use cases.

