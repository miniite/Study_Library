---
type: "practice"
difficulty: "Intermediate"
---

# Explain how a decision tree is stored as a data structure and how prediction traverses it from root to leaf.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Recursion and DP Judgment](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Recursion_and_DP_Judgment.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

Decision trees are a popular machine learning model used for both classification and regression tasks. They can be represented as a tree-like structure, where internal nodes represent features or attributes, branches represent feature values, and leaf nodes represent class labels or continuous outcomes.

#### Triaging the Question

The interviewer is likely probing your understanding of how decision trees operate at a low-level, specifically focusing on their internal representation and prediction traversal. You will need to explain the data structure used for storing the tree and describe the algorithmic process involved in making predictions.

#### First-Principles Solution
Data Structure Representation

A decision tree can be stored using a nested dictionary or an object-oriented approach. Let's consider a simple example where we have a binary decision tree (each node has two branches).

Here’s how a decision tree might be represented:

PYTHON
{
    'feature': 'temperature',
    'value': 20,
    'left_child': {
        'feature': 'humidity',
        'value': 60,
        'left_child': {'class': 'cold'},
        'right_child': {'class': 'hot'}
    },
    'right_child': {
        'feature': 'wind_speed',
        'value': 15,
        'left_child': {'class': 'sunny'},
        'right_child': {'class': 'rainy'}
    }
}
Prediction Traversal

To make a prediction, the algorithm starts at the root node and follows the branches based on the feature values provided. Let's break this down step by step:

Root Node: At each internal node (non-leaf), we compare the input value of the feature with the threshold stored in the value field.
Branch Selection: If the input value is less than or equal to the threshold, we follow the left branch; otherwise, we follow the right branch.
Leaf Node: When we reach a leaf node (a node with no children), we return the class label stored in the class field.

#### Example
Walkthrough:

Suppose we have an input vector [temperature: 18, humidity: 50]. Let's traverse the tree:

Start at the root node.

Feature: temperature, Value: 20.
Input value (temperature): 18 is less than 20, so go to the left child.

Left child of the root:

Feature: humidity, Value: 60.
Input value (humidity): 50 is less than 60, so go to the left child.

This leads us to a leaf node with class label 'cold'.

Therefore, for input [temperature: 18, humidity: 50], the prediction would be 'cold'.

#### Closing Summary

The key concept here is that decision trees are stored as nested structures where each internal node has a feature and a threshold value. Prediction involves traversing these nodes based on the input values until we reach a leaf node which provides the final class or continuous output.

#### Common Mistakes/Follow-Ups
Mistake: Not considering edge cases, such as when an input value exactly matches the threshold.

#### Follow-Up
How does the tree handle missing feature values?

#### Follow-Up
What are some optimizations for decision trees in terms of storage and prediction speed?

**Comparison:**

| Prediction | Cold | - |

**Code:**

```python
# Example Python code to represent a simple decision tree
class DecisionNode:
    def __init__(self, feature=None, value=None, left_child=None, right_child=None, class_value=None):
        self.feature = feature
        self.value = value
        self.left_child = left_child
        self.right_child = right_child
        self.class_value = class_value

# Tree structure
root_node = DecisionNode(
    feature='temperature',
    value=20,
    left_child=DecisionNode(
        feature='humidity',
        value=60,
        left_child=DecisionNode(class_value='cold'),
        right_child=DecisionNode(class_value='hot')
    ),
    right_child=DecisionNode(
        feature='wind_speed',
        value=15,
        left_child=DecisionNode(class_value='sunny'),
        right_child=DecisionNode(class_value='rainy')
    )
)
```

**Example input / output:**

```text
def predict(input_vector, node=root_node):
    if node.class_value is not None:
        return node.class_value

    if input_vector[node.feature] <= node.value:
        return predict(input_vector, node.left_child)
    else:
        return predict(input_vector, node.right_child)

input_vector = [18, 50]
prediction = predict(input_vector)
print(f"Prediction: {prediction}")
```

**Data:**

```json
{
    "temperature": 20,
    "humidity": 60,
    "wind_speed": 15
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. No
5. Yes
6. No
7. Root Node
8. Temperature <= 20?
9. Humidity <= 60?
10. class='cold'
11. class='hot'
12. Wind Speed <= 15?
13. class='sunny'
14. class='rainy'
15. Leaf Node: 'cold'

