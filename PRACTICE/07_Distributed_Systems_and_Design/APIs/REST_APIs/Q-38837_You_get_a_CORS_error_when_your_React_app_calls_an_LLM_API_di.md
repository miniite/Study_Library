---
type: "practice"
difficulty: "Intermediate"
---

# You get a CORS error when your React app calls an LLM API directly from the browser. Why does this happen and what is the right fix?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CORS](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

CORS (Cross-Origin Resource Sharing) errors occur because browsers enforce same-origin policy, which restricts how a resource loaded from one origin can interact with resources from another domain. In this case, your React app running on one domain tries to make an HTTP request directly to the LLM API hosted on a different domain, leading to a CORS error.

To fix this issue, you need to configure the server hosting the LLM API to allow requests from your React application’s origin. The right solution involves setting up proper CORS headers in the API server.

