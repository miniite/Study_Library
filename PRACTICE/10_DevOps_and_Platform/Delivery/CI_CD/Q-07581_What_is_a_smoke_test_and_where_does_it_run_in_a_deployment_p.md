---
type: "practice"
difficulty: "Intermediate"
---

# What is a smoke test and where does it run in a deployment pipeline?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Validating Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a CI/CD pipeline, a smoke test is a quick sanity check to ensure that the most critical parts of an application are functioning as expected after a new build or update. It helps catch obvious issues early in the deployment process before proceeding with more comprehensive tests.

#### Triaging the Question

The question asks about the definition and placement of smoke tests within a CI/CD pipeline. The core idea is to understand that smoke tests serve as an initial, lightweight validation step. They are typically run early in the deployment process, often right after the build stage but before more extensive integration or system-level testing.

#### First-Principles Solution
Definition: A smoke test is a set of automated checks designed to confirm that essential parts of a software application are working as expected.
Purpose: To catch simple issues quickly and prevent them from propagating further into the deployment process.
Where in the Pipeline?: Smoke tests run immediately after code has been built but before more comprehensive testing phases like unit, integration, or system-level tests.
Detailed Explanation

In a typical CI/CD pipeline, the build stage compiles and packages the new version of the application. The next step is often to run smoke tests. This ensures that the basic functionalities are not broken by any recent changes. Here’s a breakdown:

Build Stage: Source code is compiled and packaged.
Smoke Test Stage: Automated scripts or tools check if core functionalities work as expected (e.g., login, critical API endpoints).
Further Testing Stages: If smoke tests pass, the pipeline proceeds to more detailed testing phases.

#### Closing Summary

Smoke tests are crucial for maintaining the quality of deployed applications by catching simple issues early. They run right after the build stage in a CI/CD pipeline, ensuring that essential parts of the application are functional before proceeding with other tests.

#### Common Mistakes/Follow-Ups

Misplacement: Often, smoke tests can be mistaken for unit or integration tests.
Overcomplication: Sometimes, teams might over-engineer these checks, making them too complex to run quickly.
Neglecting Smoke Tests: Failing to include smoke tests can lead to significant issues being unnoticed until later stages of deployment.

**Comparison:**

```
Component	Description
Build Stage	Compiles and packages source code.
Smoke Test Stage	Runs automated checks for core functionalities.
Integration Tests	More detailed testing of the application.
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Build] --> B{Smoke Test?};
    B -- Pass --> C[Integration Tests];
    B -- Fail --> D[Error Notification];
```

**Example input / output:**

```text
Input:
- A new build is generated.
- Smoke tests are configured and set up.

Output:
- If smoke tests pass, the pipeline proceeds to integration tests.
- If they fail, an error notification is sent.
```

