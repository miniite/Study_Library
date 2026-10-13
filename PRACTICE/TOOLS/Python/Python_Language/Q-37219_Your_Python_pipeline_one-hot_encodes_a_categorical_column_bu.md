---
type: "practice"
difficulty: "Deep"
---

# Your Python pipeline one-hot encodes a categorical column but a new category appears at inference that wasn't in training. How do you handle unseen categories so the vector shape stays valid?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When handling categorical data during inference with Python, we need to ensure that the pipeline can accommodate new categories without altering the vector shape of our one-hot encoded features. This typically involves using strategies like mapping unknown categories to a specific "unknown" category or dynamically adapting the encoding process.

#### Triaging the Question

The question is about how to manage unseen categories during inference when using one-hot encoding in Python. The core concern here is maintaining consistent output dimensions regardless of new categories that appear at runtime, which could otherwise invalidate vector shapes and cause issues downstream in the pipeline.

#### First-Principles Solution
Understanding One-Hot Encoding

One-hot encoding converts categorical features into binary vectors where each category corresponds to a unique dimension. The key issue arises when an unseen category appears during inference because it creates a mismatch between training and inference, leading to vector length discrepancies.

Handling Unseen Categories: Strategies

Mapping Unknown Categories:

Create a dedicated "unknown" or "other" category in the initial pipeline setup.
During encoding, map any new categories encountered at inference to this "unknown" category.
This ensures that the output vector maintains its fixed shape and dimensionality.

Dynamic Encoding with Dictionaries:

Use dictionaries to dynamically track seen categories during both training and inference.
When a new category appears at inference, add it to the dictionary (if not already present) and map it accordingly.

Using CountVectorizer or LabelEncoder:

Libraries like CountVectorizer from scikit-learn can handle unknown categories by treating them as out-of-vocabulary words.
Alternatively, use a LabelEncoder combined with one-hot encoding to handle new categories gracefully.

#### Example
Implementation

Here’s an example using Python and the scikit-learn library:

PYTHON
from sklearn.preprocessing import OneHotEncoder, OrdinalEncoder
import numpy as np

# Training data
X_train = np.array(['cat', 'dog', 'rabbit']).reshape(-1, 1)
X_test = np.array(['cat', 'dog', 'unknown'])

# Initialize the encoder with unknown category mapping
ordinal_encoder = OrdinalEncoder()
one_hot_encoder = OneHotEncoder(sparse=False)

# Fit on training data and transform
ordinal_encoded_train = ordinal_encoder.fit_transform(X_train)
one_hot_encoded_train = one_hot_encoder.fit_transform(ordinal_encoded_train)

print("Encoded Training Data (Shape: {})".format(one_hot_encoded_train.shape))
print(one_hot_encoded_train)

# During inference, map unknown categories to the last category index
unknown_category_index = len(ordinal_encoder.categories_[0]) - 1

# Encode test data with the same ordinal encoder and handle unknowns
ordinal_encoded_test = ordinal_encoder.transform(X_test.reshape(-1, 1))
one_hot_encoded_test = one_hot_encoder.transform(ordinal_encoded_test)

# Replace any "unknown" categories in the encoding
for i in range(len(one_hot_encoded_test)):
    if ordinal_encoded_test[i] == unknown_category_index:
        one_hot_encoded_test[i] = np.zeros_like(one_hot_encoded_train[0])

print("Encoded Test Data (Shape: {})".format(one_hot_encoded_test.shape))
print(one_hot_encoded_test)

#### Closing Summary
and Common Mistakes

By ensuring that new categories are mapped to a known category or dynamically handled, we maintain consistent vector shapes in our one-hot encoded features. Common mistakes include not accounting for unseen categories, which can lead to dimensionality issues and downstream errors.

#### Follow-Ups
How would you implement this solution using pandas?
What performance considerations should be taken into account when dynamically encoding new categories during inference?

