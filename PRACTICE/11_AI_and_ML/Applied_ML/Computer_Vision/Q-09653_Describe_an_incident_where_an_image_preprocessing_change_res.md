---
type: "practice"
difficulty: "Deep"
---

# Describe an incident where an image preprocessing change (resize interpolation) silently degraded model accuracy. How would you catch this before it hits production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Image Normalization](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Image_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In computer vision, the choice of image preprocessing techniques can significantly impact model performance. An incident that comes to mind occurred during a project where we used a pre-trained CNN (Convolutional Neural Network) for object detection tasks. A change in the resize interpolation method led to a subtle yet significant drop in accuracy, which was only noticed after the model went into production.

The core of this issue lies in understanding how image preprocessing affects feature extraction and model performance. In this case, we were using bilinear interpolation, but it was changed to nearest-neighbor interpolation without thorough testing or validation.

To catch such issues before they reach production, a structured approach is essential. This incident taught us several lessons about the importance of rigorous testing, robust monitoring, and continuous improvement in our model development process.

**Comparison:**

```
Preprocessing Method	Accuracy
Bilinear Interpolation	89.5%
Nearest-Neighbor Interpolation	86.0%
```

**Example input / output:**

```text
Given the same test dataset `test_loader`, we evaluated two models with different preprocessing:
- Model 1 (bilinear interpolation): accuracy = 89.5%
- Model 2 (nearest-neighbor interpolation): accuracy = 86.0%

The difference in accuracy was not immediately obvious, but it highlighted the importance of thorough testing.
```

**Flow:**

1. Preprocess Image
2. Load Model
3. Evaluate Accuracy
4. Compare Results
5. Report Anomaly
6. Fix and Retrain
7. Re-evaluate

