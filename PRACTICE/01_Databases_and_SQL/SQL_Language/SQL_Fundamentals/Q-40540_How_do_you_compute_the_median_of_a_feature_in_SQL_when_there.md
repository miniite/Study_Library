---
type: "practice"
difficulty: "Deep"
---

# How do you compute the median of a feature in SQL when there is no built-in MEDIAN function, and why does the median matter for imputation?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the median of a feature in SQL without a built-in MEDIAN function, you need to leverage window functions like PERCENTILE_CONT. The concept revolves around finding the middle value that separates the higher half from the lower half of the data. Here’s how you can achieve this for an ML engineer with deep understanding.

To begin, let's assume we have a table named data with columns such as feature_name and value. We want to compute the median for a specific feature. The key idea is that the median minimizes the sum of absolute deviations, making it robust against outliers in skewed data distributions, which is crucial in many machine learning tasks.

