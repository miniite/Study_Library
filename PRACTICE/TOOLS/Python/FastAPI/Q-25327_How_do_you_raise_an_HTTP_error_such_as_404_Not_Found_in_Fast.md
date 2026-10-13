---
type: "practice"
difficulty: "Basic"
---

# How do you raise an HTTP error such as 404 Not Found in FastAPI?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Custom Responses](<../../../../TOOLS/Python/FastAPI/Custom_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In FastAPI, raising an HTTP error like a 404 Not Found is straightforward and can be done using the HTTPException class provided by FastAPI. This allows you to return specific HTTP status codes along with error messages, making your API more user-friendly and easier to debug.

Let's break down how this works from first principles.

