---
type: "practice"
difficulty: "Intermediate"
---

# How do you propagate build metadata (git sha, build time, version) into a container image and expose it at runtime for observability?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pinning Third-party Actions](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Pinning_Third-party_Actions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To propagate build metadata such as git SHA, build time, and version into a container image and expose them at runtime for observability, we can leverage GitHub Actions to automate the process of tagging our Docker images with this information. The key idea is to use environment variables in both your CI/CD pipeline and within the container itself.

First, let's break down what the question is asking: We need to ensure that metadata like git SHA, build time, and version are collected during the build process, stored as labels or annotations on the Docker image, and then made accessible at runtime for monitoring purposes. This involves setting up a GitHub Actions workflow that captures these pieces of information and uses them when tagging our images.

