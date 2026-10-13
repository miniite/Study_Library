---
type: "practice"
difficulty: "Basic"
---

# Why do we usually normalize pixel values from the 0-255 range to 0-1 before feeding images into a neural network?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Image Normalization](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Image_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with computer vision tasks, normalizing pixel values from the 0-255 range to 0-1 is a common preprocessing step. This process helps improve the performance and training efficiency of neural networks by ensuring that all input features are on a similar scale. In this answer, we will explore why normalization is beneficial and walk through the first-principles solution.

**Code:**

```python
# Example code for normalization
from sklearn.preprocessing import MinMaxScaler

def normalize_pixels(pixel_values):
    scaler = MinMaxScaler(feature_range=(0, 1))
    normalized_pixels = scaler.fit_transform(pixel_values)
    return normalized_pixels
```

**Example input / output:**

```text
# Example input and output
pixel_values = [[254], [128], [37]]
normalized_pixels = normalize_pixels(pixel_values)
print(normalized_pixels)
# Output:
# [[0.96862745]
#  [0.5         ]
#  [0.14509804]]
```

