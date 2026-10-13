---
type: "practice"
difficulty: "Basic"
---

# How would you accept a list of query parameters with the same name, such as ?id=1&id=2, in FastAPI?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Database Access and Tool Calls](<../../../../TOOLS/Python/FastAPI/Database_Access_and_Tool_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle multiple values for the same query parameter in FastAPI, we need to understand that by default, it will treat all provided values with the same name as a list of strings. We can leverage this behavior directly or explicitly manage the parsing logic. Here's how you would approach this:

The core idea is straightforward: when FastAPI receives multiple values for the same query parameter, such as ?id=1&id=2, it automatically converts them into a list of strings. However, we might want to parse these values further depending on our use case.

