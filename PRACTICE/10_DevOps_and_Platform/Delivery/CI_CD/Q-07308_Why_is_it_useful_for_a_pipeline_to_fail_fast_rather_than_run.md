---
type: "practice"
difficulty: "Basic"
---

# Why is it useful for a pipeline to fail fast rather than running every step even after an error?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Emergency Deploy Paths](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Emergency_Deploy_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up a CI/CD pipeline, one of the key principles is to detect and address issues as early as possible. A "fail-fast" approach in pipelines means stopping the process immediately upon encountering any failure, rather than continuing with subsequent steps. This strategy is incredibly useful because it saves time, resources, and ensures that developers are alerted quickly to problems.

To understand why fail-fast is so beneficial, let's break down what this concept entails and explore its advantages from first principles.

