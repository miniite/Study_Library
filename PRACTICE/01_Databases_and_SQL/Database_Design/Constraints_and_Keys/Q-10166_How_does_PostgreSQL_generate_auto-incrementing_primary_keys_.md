---
type: "practice"
difficulty: "Basic"
---

# How does PostgreSQL generate auto-incrementing primary keys using SERIAL or IDENTITY?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Primary and Unique Keys](<../../../../01_Databases_and_SQL/Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PostgreSQL, generating auto-incrementing primary keys can be achieved through the use of SERIAL (in older versions) and IDENTITY columns in newer versions. The core idea is that these types automatically generate a unique identifier for each row inserted into a table.

In this context, let's break down how PostgreSQL handles this using both SERIAL and IDENTITY. We'll start with the basics of what constraints and keys are and then dive into the specifics of generating auto-incrementing primary keys.

