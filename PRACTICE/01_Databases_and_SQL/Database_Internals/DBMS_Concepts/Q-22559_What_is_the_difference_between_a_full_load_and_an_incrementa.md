---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a full load and an incremental load into a warehouse table?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Schema Migration Risks](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Schema_Migration_Risks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Full load and incremental load are two common methods used in ETL (Extract, Transform, Load) processes to populate or update data warehouses. A full load involves loading all the data from source systems into the warehouse table at once, while an incremental load focuses on updating only the new or changed data since the last load.

To fully understand these concepts and their differences, let's first break down what they entail and then discuss why each method is chosen in different scenarios.

