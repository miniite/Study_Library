---
type: "practice"
difficulty: "Expert"
---

# Your fine-tuned model started producing worse outputs than the base model on some inputs. How would you use a Python eval set to detect regression and decide whether the fine-tune, the data, or the eval is at fault?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Drift-triggered Retraining](<../../../../TOOLS/Python/Python_Language/Python_Tail_Latency/Drift-triggered_Retraining.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we'll leverage an evaluation (eval) dataset in Python to systematically identify if there's a degradation in our fine-tuned model compared to its base version. The key steps involve setting up the eval set, comparing outputs, and isolating potential issues such as data drift or overfitting.

Firstly, it’s important to clarify that we're dealing with two models: the original (base) model and the newly fine-tuned model. The goal is to ensure that the fine-tuning process hasn't introduced errors or degraded performance on certain inputs.

#### Triaging the Question

The core of this question revolves around:

Identifying Degradation: Detecting whether the fine-tuned model's performance has indeed worsened.
Root Cause Analysis: Determining if the issue lies with the eval set, the data used for fine-tuning, or the evaluation process itself.

#### Assumptions
include that we have a well-defined task and a suitable eval dataset, as well as consistent metrics for comparison (e.g., accuracy, F1 score).

#### First-Principles Solution

We'll use an eval set to compare outputs from both models. Here's how the process unfolds:

Prepare the Eval Set:

Ensure the eval set is representative of the problem domain.
Use a split that mimics real-world data distribution.

Model Outputs:

For each input in the eval set, pass it through both models and record their outputs.

Performance Metrics:

Calculate performance metrics for both models (e.g., accuracy).
Compare these metrics to see if there's a significant difference.

Detailed Analysis:

If degradation is detected, analyze specific cases where the fine-tuned model performs worse.

#### Cons
ider potential issues such as data drift, overfitting, or changes in the eval set characteristics.
Step-by-Step Solution

Define Eval Set and Metrics:

PYTHON
# Define evaluation metrics
from sklearn.metrics import accuracy_score

def evaluate_model(model, X_eval, y_eval):
    predictions = model.predict(X_eval)
    return accuracy_score(y_eval, predictions)

# Load eval set
X_eval, y_eval = load_eval_dataset()

Predictions and Evaluation:

PYTHON
base_model_accuracy = evaluate_model(base_model, X_eval, y_eval)
fine_tuned_model_accuracy = evaluate_model(fine_tuned_model, X_eval, y_eval)

print(f"Base Model Accuracy: {base_model_accuracy}")
print(f"Fine-Tuned Model Accuracy: {fine_tuned_model_accuracy}")

Analyze Differences:

PYTHON
if fine_tuned_model_accuracy < base_model_accuracy:
    degraded_samples = X_eval[fine_tuned_model.predict(X_eval) != y_eval]
    print("Degraded Samples:")
    for sample in degraded_samples[:10]:
        print(sample)

Root Cause Analysis:

Data Drift: Check if the eval set is similar to training data.
Model Complexity: Ensure the fine-tuned model isn't overfitting on certain aspects of the training data.

#### Closing Summary

By systematically evaluating both models on an eval dataset, we can isolate whether the degradation is due to the evaluation process, changes in data quality or distribution, or issues with the fine-tuning strategy itself. This approach ensures that our model remains robust and aligned with real-world performance expectations.

#### Common Mistakes
and Follow-Ups
Ignoring Degradation: Failing to detect early signs of performance drops.
Inadequate Evaluation Data: Not using a representative eval set can lead to false positives or negatives.
Overfitting on Eval Set: Ensuring the eval set doesn’t include data that wasn’t seen during training.

To further refine, consider implementing more sophisticated metrics and techniques such as cross-validation or incorporating domain-specific knowledge into the evaluation process.

**Comparison:**

```
Input	Predicted Output (Base Model)	Actual Output	Predicted Output (Fine-Tuned Model)
[1, 2]	0	0	0
[3, 4]	1	1	0
[5, 6]	1	1	1
Degraded Sample Output
[3, 4]
Degraded Sample Output
[5, 6]
```

**Code:**

```python
from sklearn.metrics import accuracy_score

def evaluate_model(model, X_eval, y_eval):
    predictions = model.predict(X_eval)
    return accuracy_score(y_eval, predictions)

# Load eval set
X_eval, y_eval = load_eval_dataset()

base_model_accuracy = evaluate_model(base_model, X_eval, y_eval)
fine_tuned_model_accuracy = evaluate_model(fine_tuned_model, X_eval, y_eval)

print(f"Base Model Accuracy: {base_model_accuracy}")
print(f"Fine-Tuned Model Accuracy: {fine_tuned_model_accuracy}")

if fine_tuned_model_accuracy < base_model_accuracy:
    degraded_samples = X_eval[fine_tuned_model.predict(X_eval) != y_eval]
    print("Degraded Samples:")
    for sample in degraded_samples[:10]:
        print(sample)
```

**Example input / output:**

```text
# Example input -> output of the above code snippet

# Loading eval set
X_eval, y_eval = load_eval_dataset()

# Evaluating models
base_model_accuracy = evaluate_model(base_model, X_eval, y_eval)
fine_tuned_model_accuracy = evaluate_model(fine_tuned_model, X_eval, y_eval)

print(f"Base Model Accuracy: {base_model_accuracy}")
print(f"Fine-Tuned Model Accuracy: {fine_tuned_model_accuracy}")

# If fine-tuned model accuracy is lower
if base_model_accuracy > fine_tuned_model_accuracy:
    degraded_samples = X_eval[fine_tuned_model.predict(X_eval) != y_eval]
    print("Degraded Samples:")
    for sample in degraded_samples[:10]:
        print(sample)
```

**Data:**

```python
# Example eval dataset loading function (dummy implementation)
def load_eval_dataset():
    # Placeholder data
    X_eval = [[1, 2], [3, 4], [5, 6]]
    y_eval = [0, 1, 1]
    return X_eval, y_eval
```

**Flow:**

1. No
2. Yes
3. Load Eval Set
4. Evaluate Base Model
5. Accuracy > Threshold?
6. Evaluate Fine-Tuned Model
7. Fine-Tuned Accuracy < Base?
8. Analyze Degraded Samples
9. Isolate Root Cause (Data Drift,Overfitting)
10. Fine-Tuned Model Accuracy<= Base Model Accuracy
11. Degradation Detected

