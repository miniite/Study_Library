---
type: "practice"
difficulty: "Basic"
---

# What is a 'stage' in a pipeline, and can you name a sensible order of stages for shipping a monitoring change?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ CI CD Pipeline Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_CD_Pipeline_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In the context of Continuous Integration/Continuous Deployment (CI/CD), a "stage" represents a distinct phase or checkpoint within a pipeline where specific operations are performed on the code or artifacts. These stages ensure that changes go through various tests and validations before they reach production, maintaining quality and reliability.

The typical order of stages in a CI/CD pipeline for shipping a monitoring change is: Source Control -> Build -> Test -> Deploy -> Monitor. Each stage has its own purpose:

Source Control: This is where developers store their code. Staging environments often have a repository that mirrors the production environment but allows changes to be tested before they are merged into main.
Build: Here, the source code is compiled and packaged into an executable format. Tools like Jenkins or GitHub Actions can automate this process.
Test: The built artifacts go through various tests such as unit tests, integration tests, and end-to-end tests to ensure that the change does not introduce bugs and meets all requirements.
Deploy: Once tested successfully, the artifacts are deployed to a staging environment for further validation in a production-like setup.
Monitor: Finally, the change is monitored closely in the staging environment to check its performance, stability, and any potential issues.

**Code:**

```python
# Example of a simple Jenkinsfile snippet defining stages

pipeline {
    agent any
    stages {
        stage('Source Control') { ... }
        stage('Build') { ... }
        stage('Test') { ... }
        stage('Deploy') { ... }
        stage('Monitor') { ... }
    }
}
```

