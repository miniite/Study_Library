---
type: "practice"
difficulty: "Basic"
---

# How would you enable CORS on your Flask LLM API so a React chat frontend on a different origin can call it?

**Difficulty:** Basic · **Levels:** Fresher · [↑ CORS](<../../../../TOOLS/Python/Flask/CORS.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To allow a React chat frontend running on a different domain (origin) to make requests to an AI LLM API built with Flask, we need to enable Cross-Origin Resource Sharing (CORS). CORS is crucial because it helps us control which domains can access our backend APIs. In this scenario, we want the Flask API to accept requests from the React frontend's domain.

To implement this, we'll use the flask-cors extension for Flask. This package simplifies adding CORS support and allows us to define the origins that are allowed to make cross-origin requests.

