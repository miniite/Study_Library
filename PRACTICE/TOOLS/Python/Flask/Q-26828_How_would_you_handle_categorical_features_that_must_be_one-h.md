---
type: "practice"
difficulty: "Deep"
---

# How would you handle categorical features that must be one-hot encoded consistently between training and a Flask serving path?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Loading Models Efficiently](<../../../../TOOLS/Python/Flask/Loading_Models_Efficiently.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to manage one-hot encoding of categorical features consistently between training and a Flask serving path, we need to ensure that both processes use the same vocabulary or set of categories. This involves creating a pipeline in our ML model that standardizes this process during training, and then deploying it within a Flask application so that it can be used for real-time predictions. Here’s a step-by-step breakdown from first principles:

First, let's understand what one-hot encoding is: It converts categorical features into binary vectors where each category corresponds to a unique vector. For example, if we have a feature 'color' with categories ['red', 'green', 'blue'], it will be encoded as:

['red'] -> [1, 0, 0]
['green'] -> [0, 1, 0]
['blue'] -> [0, 0, 1]

The challenge is to ensure that the same encoding scheme used during training is applied when serving predictions via Flask. We'll use scikit-learn’s OneHotEncoder for this purpose.

Let's dive into the implementation:

Prepare and Encode Categorical Data During Training:

Use OneHotEncoder from scikit-learn to encode categorical features during training.
Fit the encoder on the training data and transform it.

Store the Encoder for Serving:

Save the fitted OneHotEncoder so that it can be used at serving time without retraining.

Deploy Flask API with Consistent Encoding:

Use Flask to create an API that encodes incoming categorical features using the stored encoder.
Ensure the model in your Flask application uses the same encoding scheme.

Now, let's illustrate this process with some code and a brief explanation:

