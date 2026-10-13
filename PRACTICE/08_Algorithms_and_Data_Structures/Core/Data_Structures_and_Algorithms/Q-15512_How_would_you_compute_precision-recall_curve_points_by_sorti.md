---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute precision-recall curve points by sorting scores once, and why does one pass suffice after sorting?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Inverted Indexes](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Inverted_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the precision-recall curve points efficiently, we can leverage a single sorted array of scores to minimize redundant operations. This approach ensures that we only traverse the list once, making it an O(n) solution where n is the number of data points. Let's break down how this works step by step.

#### Triage
Understanding the Question and What’s Being Probed

The core of the question revolves around optimizing the computation of precision-recall curve points. Precision measures the accuracy of positive predictions, while recall measures the completeness of all actual positives. Sorting the scores once is a key optimization to reduce the computational complexity from O(n^2) or higher to O(n). The interviewer is probing your understanding of algorithmic efficiency and your ability to leverage data structures for optimization.

#### First-Principles Solution
Step-by-Step Breakdown
Sorting: Start by sorting all the scores in descending order. This allows us to process each score from highest to lowest, making it easier to track true positives (TP) and false positives (FP).
Initialization: Initialize counters for TP, FP, and recall. Also, initialize a variable to keep track of the current number of positive predictions.
Traversal: Traverse through the sorted scores:
For each score, increment the total number of positive predictions.
Check if this prediction is a true positive (TP) or false positive (FP). This can be done by comparing with an actual label array.
Update Precision and Recall: Calculate precision as TP / (TP + FP) at each step. Update recall using the current count of positive predictions.
Full Solution: Implementation

Here’s how you would implement this in Python:

PYTHON
def compute_precision_recall_curve(scores, labels):
    # Step 1: Sort scores in descending order
    sorted_indices = sorted(range(len(scores)), key=lambda k: -scores[k])

    tp = 0  # True positives
    fp = 0  # False positives
    total_positives = sum(labels)  # Total number of actual positive cases

    # Initialize a list to store precision and recall values
    precision_recall_points = []

    for i in sorted_indices:
        if labels[i] == 1:  # If the label is positive, it's a true positive
            tp += 1
        else:  # Otherwise, it’s a false positive
            fp += 1

        current_precision = tp / (tp + fp) if (tp + fp) > 0 else 0
        precision_recall_points.append((current_precision, tp / total_positives))

    return precision_recall_points

# Example usage:
scores = [0.95, 0.85, 0.75, 0.65, 0.55, 0.45]
labels = [1, 1, 1, 0, 0, 1]

points = compute_precision_recall_curve(scores, labels)
print(points)

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that the precision-recall curve is computed efficiently by leveraging a single pass through the sorted scores. Key mistakes to avoid include:

Not sorting the scores, leading to higher complexity.
Miscounting true positives and false positives, which can skew the results.

Common follow-ups might involve discussing how this method scales with larger datasets, or considering edge cases like when all predictions are negative or identical scores.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
    title Compute Precision-Recall Curve Points
    section Sorting and Initialization
        sorted_indices: t=1
        initialization: t=2
    section Traversal and Calculation
        for i in range(len(sorted_indices)):
            tp += 1 if labels[i] == 1 else fp += 1
            precision = tp / (tp + fp) if (tp + fp) > 0 else 0
            recall = tp / total_positives
```

**Code:**

```python
def compute_precision_recall_curve(scores, labels):
    # Step 1: Sort scores in descending order
    sorted_indices = sorted(range(len(scores)), key=lambda k: -scores[k])

    tp = 0  # True positives
    fp = 0  # False positives
    total_positives = sum(labels)  # Total number of actual positive cases

    # Initialize a list to store precision and recall values
    precision_recall_points = []

    for i in sorted_indices:
        if labels[i] == 1:  # If the label is positive, it's a true positive
            tp += 1
        else:  # Otherwise, it’s a false positive
            fp += 1

        current_precision = tp / (tp + fp) if (tp + fp) > 0 else 0
        precision_recall_points.append((current_precision, tp / total_positives))

    return precision_recall_points

# Example usage:
scores = [0.95, 0.85, 0.75, 0.65, 0.55, 0.45]
labels = [1, 1, 1, 0, 0, 1]

points = compute_precision_recall_curve(scores, labels)
print(points)
```

**Example input / output:**

```text
[(1.0, 0.5), (0.75, 0.6666666666666666), (0.5, 0.8333333333333334)]
```

**Data:**

```json
{
    "scores": [0.95, 0.85, 0.75, 0.65, 0.55, 0.45],
    "labels": [1, 1, 1, 0, 0, 1]
}
```

**Flow:**

1. Yes
2. No
3. Not done
4. Done
5. Start
6. Sort scores in descendingorder
7. Initialize counters for TP, FPand total positives
8. Traverse sorted indices
9. Is label[i] == 1?
10. Increment TP
11. Increment FP
12. F,G
13. Calculate current precision andrecall
14. Store (precision, recall) inpoints
15. Check if all indices areprocessed
16. End

