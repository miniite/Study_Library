---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between spark.read.parquet lazy schema inference and providing an explicit schema — why does explicit schema matter for cost and correctness?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Schema_Evolution_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PySpark, when you read a Parquet file, you can either rely on lazy schema inference or provide an explicit schema. Lazy schema inference is the default behavior where Spark infers the schema from the data at runtime. Providing an explicit schema involves defining the schema before reading the data, which allows for more control and potentially optimization.

The key difference lies in cost and correctness. Explicitly specifying a schema can lead to performance benefits and ensure correctness, but it requires additional effort upfront.

