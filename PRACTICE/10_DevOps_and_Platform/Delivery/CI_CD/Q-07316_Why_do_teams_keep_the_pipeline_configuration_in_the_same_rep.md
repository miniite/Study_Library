---
type: "practice"
difficulty: "Basic"
---

# Why do teams keep the pipeline configuration in the same repo as the code (pipeline-as-code)?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When teams adopt a DevOps culture, keeping the pipeline configuration within the same repository as the application code is known as "pipeline-as-code" or "PaC." This approach integrates continuous integration and delivery (CI/CD) pipelines directly into source control management (SCM), enabling more efficient and collaborative development. By placing both the code and its associated CI/CD configurations in a single repository, teams can manage their pipelines alongside their application code, making changes easier to track and version.

**Code:**

```yaml
# Example of a simple pipeline as code configuration using Jenkinsfile
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
            }
        }
    }
}
```

**Flow:**

1. Code Repository
2. (CI/CD Pipeline Configuration)
3. Application Code
4. Automated Build andDeployment

