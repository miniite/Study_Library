---
type: "practice"
difficulty: "Basic"
---

# How would you enable CORS in a FastAPI GenAI service so a browser-based chat frontend on another origin can call it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ WebSockets in FastAPI](<../../../../TOOLS/Python/FastAPI/WebSockets_in_FastAPI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enable Cross-Origin Resource Sharing (CORS) in a FastAPI GenAI service, we need to configure our FastAPI application to allow requests from specific origins. This is crucial because by default, browsers enforce the Same Origin Policy, which restricts web pages from making requests to a different domain than that of the site served on the page.

In this scenario, enabling CORS means allowing a browser-based chat frontend running on another origin (e.g., https://frontend.example.com) to make HTTP requests to your FastAPI GenAI service. Let's break down how we can achieve this from first principles.

