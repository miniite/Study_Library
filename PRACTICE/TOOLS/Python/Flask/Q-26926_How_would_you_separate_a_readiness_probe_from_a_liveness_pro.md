---
type: "practice"
difficulty: "Deep"
---

# How would you separate a readiness probe from a liveness probe for a Flask RAG service that loads the vector index slowly at startup?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Long-running Work in Flask](<../../../../TOOLS/Python/Flask/Long-running_Work_in_Flask.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To separate a readiness probe from a liveness probe in a Flask RAG (Retrieval-Augmented Generation) service, we need to understand what each probe checks and how they are used in Kubernetes deployments. A readiness probe ensures that the service is ready to accept traffic, while a liveness probe verifies that the service is still running and healthy. In this context, the vector index loading at startup will affect the service's readiness but not its liveness.

#### Triage
The question is asking how to configure these probes in a Flask application context where the vector index takes time to load during startup. By setting up separate probes for each case, we can ensure that the deployment waits until the vector index is fully loaded before marking the service as ready to handle traffic.

