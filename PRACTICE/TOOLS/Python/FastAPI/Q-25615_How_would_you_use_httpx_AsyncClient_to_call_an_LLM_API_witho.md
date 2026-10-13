---
type: "practice"
difficulty: "Intermediate"
---

# How would you use httpx AsyncClient to call an LLM API without blocking the event loop in an async FastAPI endpoint?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Async vs Sync Endpoints](<../../../../TOOLS/Python/FastAPI/Async_vs_Sync_Endpoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll discuss how to integrate httpx.AsyncClient into a FastAPI application to make non-blocking HTTP calls to an LLM API from within an asynchronous endpoint. This involves understanding FastAPI's async capabilities and the httpx library, which is designed for making efficient HTTP requests.

The primary goal here is to ensure that the event loop remains unblocked when we call an external API, allowing other tasks to continue running concurrently. We'll explore how to set up this integration step-by-step, starting from defining the endpoint in FastAPI and moving through to the actual implementation of the httpx.AsyncClient.

