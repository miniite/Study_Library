---
type: "practice"
difficulty: "Intermediate"
---

# What does ON DELETE RESTRICT or NO ACTION do, and how do they differ from CASCADE?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Referential Actions](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Referential_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

ON DELETE RESTRICT or NO ACTION in database constraints prevent you from deleting a record if it is still referenced by another table, which ensures data integrity but can be restrictive. CASCADE allows the deletion to propagate through related tables automatically. Let's dive into these concepts and their differences.

Confirmed: The answer covers the basics of ON DELETE RESTRICT and NO ACTION, contrasts them with CASCADE, and provides a clear explanation.

