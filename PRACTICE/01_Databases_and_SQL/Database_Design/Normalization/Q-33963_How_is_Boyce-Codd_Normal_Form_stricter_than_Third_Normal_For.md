---
type: "practice"
difficulty: "Deep"
---

# How is Boyce-Codd Normal Form stricter than Third Normal Form?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ BCNF vs 3NF](<../../../../01_Databases_and_SQL/Database_Design/Normalization/BCNF_vs_3NF.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database normalization, Boyce-Codd Normal Form (BCNF) is a more stringent form compared to Third Normal Form (3NF). BCNF requires that every determinant be a candidate key, making it stricter in handling transitive dependencies and ensuring data integrity. This question asks us to understand the strictness of BCNF over 3NF by exploring their definitions, differences, and why BCNF is considered more rigorous.

**Comparison:**

| BCNF vs 3NF | Third Normal Form (3NF)                      | Boyce-Codd Normal Form (BCNF)                       |
|-------------|---------------------------------------------|----------------------------------------------------|
| Definition  | Eliminate transitive dependencies           | Every determinant must be a candidate key          |
| Example     | Employee table with department and salary   | Order table where order_id and customer_id are keys |

