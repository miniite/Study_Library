---
type: "practice"
difficulty: "Deep"
---

# Your observability-pipeline repo holds Vector, Fluent Bit, and OpenTelemetry-collector configs, and a merged pipeline change silently dropped a label that a downstream alert grouped on. How would a required config-contract test on the PR have caught the dropped label before it reached the collector fleet?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolating CI Environments](<../../../../TOOLS/Git_and_GitHub/Git_Core/Isolating_CI_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with an observability pipeline where Vector, Fluent Bit, and OpenTelemetry-collector configurations are managed in a Git repository (e.g., GitHub). A recently merged PR inadvertently removed a label that is crucial for downstream alerts to function correctly. We need to discuss how implementing a config-contract test on the PR could have prevented this issue.

The core idea here is to enforce configuration consistency and completeness through automated tests, ensuring that any changes do not introduce broken or missing labels. Let's break down the steps involved in setting up such a test from first principles.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Repo] --> B[CI/CD Trigger];
    B --> C[Pre-Commit Hook];
    C --> D[Config Validation Script];
    D --> E[Pass/Fail];
```

**Flow:**

1. Repo
2. CI/CD Trigger
3. Pre-Commit Hook
4. Config Validation Script
5. Pass/Fail

