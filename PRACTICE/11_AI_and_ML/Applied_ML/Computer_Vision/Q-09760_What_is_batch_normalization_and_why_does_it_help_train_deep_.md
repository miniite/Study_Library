---
type: "practice"
difficulty: "Intermediate"
---

# What is batch normalization and why does it help train deep convolutional networks?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Image Normalization](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Image_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Batch normalization is a technique used in training neural networks to normalize the input of each layer, thus stabilizing the learning process and accelerating convergence. In the context of computer vision, where models often have many layers and suffer from internal covariate shift (the changing distribution of inputs due to weight updates), batch normalization helps mitigate this issue. Here’s a step-by-step explanation of how it works and why it's beneficial.

