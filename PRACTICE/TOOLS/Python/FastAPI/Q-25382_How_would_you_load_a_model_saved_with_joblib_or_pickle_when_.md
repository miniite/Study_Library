---
type: "practice"
difficulty: "Basic"
---

# How would you load a model saved with joblib or pickle when your FastAPI application starts?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Packaging Models in Containers](<../../../../TOOLS/Python/FastAPI/Packaging_Models_in_Containers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To load a model saved with joblib or pickle into a FastAPI application, we need to ensure that the model is properly loaded during the startup of our server. This involves several steps: setting up the FastAPI app, defining an endpoint to serve predictions, and ensuring the model loads before any incoming requests are processed.

The key idea here is to leverage FastAPI's lifecycle hooks to load the model when the application starts. We'll use the startup event handler provided by FastAPI to handle this task. This approach ensures that our model is ready for use as soon as the server is up and running.

