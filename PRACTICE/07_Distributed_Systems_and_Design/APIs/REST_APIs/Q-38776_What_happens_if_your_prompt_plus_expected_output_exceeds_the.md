---
type: "practice"
difficulty: "Intermediate"
---

# What happens if your prompt plus expected output exceeds the model's context window, and what status or error does the LLM REST API return?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling 429 Responses](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Handling_429_Responses.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a prompt plus its expected output exceeds the model's context window in an AI system using a REST API, several things can happen depending on how the system is designed. Typically, the API will return a specific error or status code to indicate that the request was invalid due to exceeding the context limit. This response helps both the client and server handle such cases gracefully.

