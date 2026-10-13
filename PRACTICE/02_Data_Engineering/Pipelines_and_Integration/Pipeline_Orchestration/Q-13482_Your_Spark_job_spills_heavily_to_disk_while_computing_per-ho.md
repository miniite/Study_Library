---
type: "practice"
difficulty: "Deep"
---

# Your Spark job spills heavily to disk while computing per-host metric percentiles; what levers do you pull first before adding memory?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Profiling Spark Stages](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Profiling_Spark_Stages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with heavy disk spilling in a Spark job, especially when computing per-host metric percentiles, it's essential to diagnose and optimize from multiple angles rather than immediately adding more memory. The key is understanding where the bottlenecks lie—whether they are data shuffling issues, IO constraints, or processing inefficiencies.

To effectively address this issue, we should first:

Diagnose Data Shuffling: Evaluate if the shuffle phase is causing excessive disk usage.
Profile Memory Usage: Check how memory is being utilized and identify potential areas of optimization.
Optimize Partitioning and Co-location: Ensure data locality to reduce shuffles and improve job performance.
Tune Shuffle Settings: Adjust settings that control shuffling behavior, such as spark.shuffle.service.enabled.
Improve Data Transformation Logic: Look into the transformations used in your Spark code for any inefficiencies.

Let's delve deeper into each step.

