---
type: "practice"
difficulty: "Basic"
---

# How would you write a Pydantic response model for a chat endpoint that returns the answer plus token usage and model name?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/FastAPI/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we'll be using FastAPI, a modern web framework for building APIs with Python 3.7+ and above, along with Pydantic, which is used to define models that validate and serialize data. Our goal is to create a response model that includes the answer from a chat endpoint, token usage, and the name of the model.

The question essentially asks us to design a Pydantic response model in FastAPI for a chat endpoint. Here's how we can approach this step-by-step:

Understand the Question: We need to create a Pydantic model that encapsulates the data structure expected from our API response.
Define Assumptions: Assume the answer is just a string, token usage is an integer, and model name is also a string.

Let's proceed with these assumptions and build the model in FastAPI.

