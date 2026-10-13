---
type: "practice"
difficulty: "Intermediate"
---

# What is the IQR method for detecting outliers and how do you compute the bounds?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Seasonal Baselines](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Seasonal_Baselines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To detect outliers in a dataset, one common method is using the Interquartile Range (IQR). The IQR method involves calculating the first quartile (Q1) and third quartile (Q3), then determining the interquartile range (IQR = Q3 - Q1). Outliers are typically defined as data points that fall below (Q1 - 1.5 \times \text{IQR}) or above (Q3 + 1.5 \times \text{IQR}).

In this answer, we'll break down the IQR method step by step and discuss its implementation in Python.

