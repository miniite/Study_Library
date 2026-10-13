---
type: "practice"
difficulty: "Deep"
---

# How would you diagnose a Spark SQL job where one task runs 10x longer than the rest? Walk through reading the stage's task-duration distribution and identifying skew versus a slow node.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose a Spark SQL job where one task runs 10x longer than the rest, we need to understand whether the issue is due to data skew or if there's a single underperforming node. Let's walk through this step-by-step.

First, we should start by examining the stage's task-duration distribution and identifying any obvious signs of skew or slow nodes.

