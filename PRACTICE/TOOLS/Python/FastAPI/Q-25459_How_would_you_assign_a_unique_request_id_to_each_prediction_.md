---
type: "practice"
difficulty: "Intermediate"
---

# How would you assign a unique request id to each prediction call in FastAPI so you can trace it in logs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Database Access and Tool Calls](<../../../../TOOLS/Python/FastAPI/Database_Access_and_Tool_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that we can trace predictions in logs, assigning a unique request ID to each prediction call is essential. In this scenario, I'll use FastAPI, an async web framework for building APIs with Python 3.7+ (and beyond). The key idea here is to generate a unique identifier and associate it with the incoming request so that we can correlate log entries across different parts of our application.

FastAPI allows us to easily attach metadata to requests by using context variables or middleware, which will enable us to track the request ID throughout the prediction process. Let's break down how to implement this from first principles.

