---
type: "practice"
difficulty: "Basic"
---

# What is a pipeline-as-code approach and how does it differ from clicking through a UI to configure jobs?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a CI/CD (Continuous Integration/Continuous Delivery) context, pipeline-as-code refers to defining your build, test, and deployment processes using code rather than manually configuring them via a user interface. This approach leverages version control systems like Git, making pipelines more maintainable, reusable, and easier to share across teams. It contrasts with the traditional method of manually setting up jobs through an interface, which can be error-prone and difficult to manage as team size and complexity grow.

**Code:**

```yaml
# Example Jenkinsfile for a simple CI/CD pipeline in YAML format

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                sh 'make test'
            }
        }
        stage('Deploy') {
            when { expression { return env.BRANCH_NAME == 'main' } }
            steps {
                sh 'make deploy'
            }
        }
    }
}
```

**Flow:**

1. Source Code
2. Commit to Repository
3. CI Server Triggers Build onCommit
4. Run Tests and Build Artifacts
5. If Successful, Proceed toDeploy
6. Deploy to StagingEnvironment
7. Manual Approval forProduction Deployment
8. Automated Rollback if IssuesFound

