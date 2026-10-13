---
type: "practice"
difficulty: "Basic"
---

# What is a pipeline stage versus a job versus a step?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI CD Pipeline Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_CD_Pipeline_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing CI/CD pipelines, it's important to understand the different components that make up these systems. A pipeline can be broken down into stages, which are high-level segments of your workflow; jobs are individual tasks within a stage; and steps are the smallest units of work that perform specific actions.

A pipeline stage is like a phase or milestone in your CI/CD process, such as "Build" or "Test". Each stage represents a significant step in the development lifecycle. For example, you might have a "Build" stage where code is compiled, and a "Deploy" stage where changes are deployed to production.

A job encompasses all the actions that take place within a specific stage. For instance, if your "Build" stage includes multiple tasks like compiling code and running tests, each of these would be considered separate jobs. In Jenkins terminology, for example, you might have a job named build-app which triggers the build process.

A step is the most granular unit within a job; it's a specific command or action that performs a task. For example, inside the "build-app" job, there could be steps such as mvn clean install for compiling Java code and npm test for running JavaScript tests.

