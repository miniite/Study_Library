---
type: "practice"
difficulty: "Basic"
---

# How would you implement early stopping with patience, and what small data structure tracks the best-so-far metric and counter?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Sliding Window Aggregates](<../../../../08_Algorithms_and_Data_Structures/Core/Data_Structures_and_Algorithms/Sliding_Window_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Early stopping is a crucial technique used to prevent overfitting in machine learning models by monitoring a validation metric during training and terminating early if the performance does not improve. The "patience" parameter specifies how many epochs without improvement should occur before we stop the training.

To implement this, we need two key components: a data structure to track the best-so-far metric and a counter for tracking the number of consecutive non-improving epochs. A simple way to achieve this is by using a dictionary to store the best metric and an integer variable for the patience counter.

Here’s how we can break down the solution:

Initialization: Start with initializing variables that will hold the best-so-far metric.
Tracking Improvement: During each epoch, check if the current validation metric improves upon the best-so-far metric.
Updating Best Metric and Counter: If the metric does improve, update the best metric and reset the counter to zero. Otherwise, increment the counter by one.
Stopping Condition: Training stops when the counter reaches the patience threshold.

Let’s go through this step-by-step using simple foundational concepts.

