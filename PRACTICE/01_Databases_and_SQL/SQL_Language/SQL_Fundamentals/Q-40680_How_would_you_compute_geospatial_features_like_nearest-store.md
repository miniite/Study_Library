---
type: "practice"
difficulty: "Deep"
---

# How would you compute geospatial features like nearest-store distance in SQL for a delivery-time model, and when do you prefer a bounding-box prefilter over full haversine over all rows?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Vector Similarity in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Vector_Similarity_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute geospatial features such as the nearest store distance in SQL for a delivery-time model, we need to consider several factors including performance, accuracy, and complexity. The two primary methods are using bounding-box prefiltering (also known as spatial indexing) versus calculating the exact distance using the Haversine formula or similar. Each method has its trade-offs and is preferred based on specific requirements.

**Comparison:**

```
Feature	Bounding-Box Prefiltering	Haversine Calculation
Performance	Faster due to reduced rows for full calculation	Slower as it needs to calculate distances for all rows
Accuracy	Lower accuracy but sufficient in most cases	High accuracy, best for precise distance calculations
Complexity	Easier to implement and maintain	More complex implementation with potentially higher computational cost
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Bounding-Box PrefilteringSufficient?
5. BoundingBoxPrefilter
6. HaversineCalculation
7. Return Closest Store
8. Calculate Exact Distance UsingHaversine
9. Return Closest Store
10. End

