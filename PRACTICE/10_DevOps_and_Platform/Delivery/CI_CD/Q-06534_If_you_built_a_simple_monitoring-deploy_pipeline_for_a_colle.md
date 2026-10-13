---
type: "practice"
difficulty: "Basic"
---

# If you built a simple monitoring-deploy pipeline for a college or side project, what one improvement would you add next and why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Introducing CI CD to Legacy Systems](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Introducing_CI_CD_to_Legacy_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When building a simple CI/CD pipeline for a college or side project, it's essential to focus on making the process robust yet minimalistic. The first step is often setting up basic monitoring and deploying changes efficiently. After those basics are in place, the next improvement I would suggest is adding automated rollback capabilities. This ensures that if a deployment goes wrong, you can quickly revert to the previous stable state without manual intervention.

The core idea here is to add robust error handling and recovery mechanisms into your CI/CD pipeline. Let's break down why this is important and how we can implement it.

