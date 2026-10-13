---
type: "practice"
difficulty: "Basic"
---

# How would you preprocess a raw duration string like 250ms or 1.2s into a consistent numeric unit for latency analysis?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Cleaning Text and Numeric Fields](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To preprocess and clean a raw duration string such as "250ms" or "1.2s" into a consistent numeric value, we need to break down the problem step by step from first principles. The core idea is to accurately convert these strings into seconds (or another common unit) for consistency in latency analysis. We'll start with understanding the input format, then parse and extract the numerical part along with its corresponding time unit, convert it to a consistent numeric value, and finally handle any edge cases or errors that might arise.

