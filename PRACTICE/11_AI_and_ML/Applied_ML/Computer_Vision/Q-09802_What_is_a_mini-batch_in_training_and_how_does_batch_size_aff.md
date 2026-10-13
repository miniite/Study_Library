---
type: "practice"
difficulty: "Intermediate"
---

# What is a mini-batch in training and how does batch size affect memory and gradient stability for image models?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Image Normalization](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Image_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In machine learning, particularly within computer vision, a mini-batch refers to a subset of data from the dataset used to compute the model's gradients during training. The batch size is the number of samples in each mini-batch. This approach is crucial for balancing computational efficiency and gradient estimation accuracy. In this explanation, we'll explore how mini-batches work, their impact on memory usage, and the effect on gradient stability.

**Comparison:**

```
Batch Size	Memory Usage	Gradient Stability
Small	Low	High
Medium	Balanced	Moderate
Large	High	Low
```

**Code:**

```python
# Example code to define a mini-batch
def get_mini_batch(X, y, batch_size=32):
    indices = np.random.permutation(len(X))  # Shuffle the data
    for i in range(0, len(X), batch_size):
        batch_indices = indices[i:i + batch_size]
        yield X[batch_indices], y[batch_indices]

# Example usage of a mini-batch generator
X_train = np.array([[1, 2], [3, 4], [5, 6], [7, 8]])
y_train = np.array([0, 1, 0, 1])
mini_batch_generator = get_mini_batch(X_train, y_train, batch_size=2)
for X_batch, y_batch in mini_batch_generator:
    print(f'Batch: {X_batch}, Labels: {y_batch}')
```

**Example input / output:**

```text
Batch: [[1 2] [3 4]], Labels: [0 1]
Batch: [[5 6] [7 8]], Labels: [0 1]
```

**Data:**

```json
{
  "X_train": [
    [1, 2],
    [3, 4],
    [5, 6],
    [7, 8]
  ],
  "y_train": [0, 1, 0, 1]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Define Mini-Batch
6. Is Batch Size Small?
7. Memory Usage Low, GradientStability High
8. Is Batch Size Large?
9. Memory Usage High, GradientStability Low
10. Memory Usage Balanced,Gradient Stability Moderate

