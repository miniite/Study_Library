---
type: "practice"
difficulty: "Basic"
---

# How would you explain to a non-technical teammate why a report query is slow after looking at its plan?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Diagnosing Query Regressions](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Diagnosing_Query_Regressions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To a non-technical teammate, I’d start by explaining that SQL queries are like instructions for a recipe—just as a chef needs to follow a well-thought-out recipe to make sure the dish turns out right and doesn’t take too long to prepare, our database needs to process the query efficiently. When we look at the execution plan of a slow report query, it’s like peeking behind the scenes to see exactly how the database is handling the request.

The execution plan shows us step-by-step what the database does when it runs the query. It helps us understand why the query might be taking longer than expected and identify potential bottlenecks. By understanding this plan, we can make adjustments that speed up the process, similar to finding ways to optimize a recipe for better cooking time.

