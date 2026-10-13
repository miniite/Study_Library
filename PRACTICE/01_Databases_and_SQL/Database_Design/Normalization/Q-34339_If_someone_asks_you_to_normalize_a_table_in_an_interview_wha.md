---
type: "practice"
difficulty: "Basic"
---

# If someone asks you to normalize a table in an interview what is the very first thing you would check about its columns?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Redundancy and Anomalies](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Redundancy_and_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When normalizing a table, the very first thing I would check is whether each column contains atomic data—meaning that it represents a single piece of information without duplication or redundancy. This is crucial because normalization aims to eliminate repeating groups and ensure data integrity.

To delve into this in more detail:

Identify Redundant Data: Look for columns that contain duplicate values, especially if these duplicates appear across multiple rows.
Check for Functional Dependencies: Determine whether a column’s value can be derived from another column or combination of columns. If so, consider combining those dependencies to reduce redundancy.

Here's how I would approach this step-by-step:

List All Columns and Their Values - For instance, if we have a table Orders with columns: OrderID, CustomerID, ProductID, Quantity, and DateOrdered.
Identify Redundant Rows: Check for repeated values in non-key columns. If the same set of values appears across multiple rows (e.g., different Orders having the same CustomerID or ProductID), this suggests redundancy.
Analyze Functional Dependencies: For example, if DateOrdered can be derived from a combination of other columns (like OrderID and CustomerID), it might not need to be stored separately.

This initial check sets the foundation for understanding how to proceed with normalization properly.

