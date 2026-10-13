---
type: "practice"
difficulty: "Deep"
---

# What is progressive delivery and how does it relate to GitOps?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Progressive delivery is a set of practices that enable organizations to gradually roll out new software changes in production, ensuring they are stable before fully deploying them to all users. It aligns with GitOps by leveraging Git as the single source of truth and using automation tools to maintain consistency between environments.

The key idea behind progressive delivery involves split testing and canary releases to validate changes incrementally. This approach minimizes risk and allows teams to make more informed decisions about whether a change is ready for full-scale deployment. Now, let's dive into how GitOps supports these practices by ensuring that the configuration of your system is version-controlled in Git.

First, we need to understand what GitOps is: it’s an operational pattern where infrastructure as code (IaC) is stored in a Git repository and maintained using continuous integration/continuous deployment (CI/CD). The goal is to ensure that the production environment reflects the state defined in Git. This consistency across environments reduces the chances of divergence.

To relate progressive delivery with GitOps, we start by setting up a CI/CD pipeline where each commit to the main branch triggers an automated build and deploy process. These processes can be fine-tuned to perform split testing or canary releases:

Split Testing: A small percentage of users are directed to the new version while the majority continue using the old one. This helps gather feedback and monitor application performance before a full-scale rollout.
Canary Releases: Similar to split testing, but often used for more targeted groups (e.g., by region or user role). It’s particularly useful when dealing with complex systems where you want to test in a controlled manner.

By integrating these practices within a GitOps framework, teams can ensure that every change is tracked and tested systematically. The automation of the deployment process ensures consistency between development and production environments, reducing human error and allowing for faster feedback loops.

In summary, progressive delivery enhances reliability by introducing staged rollouts, while GitOps provides a robust foundation for managing these changes through version-controlled infrastructure. This integration allows teams to iterate quickly with confidence.

**Comparison:**

```
Stage	Action
Build	Run Maven clean install
Test	Run Maven tests
Deploy	Apply IaC changes with Flux
```

**Code:**

```bash
# Example CI/CD pipeline script snippet using Jenkins
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    // Use a tool like Flux to sync the GitOps configuration
                    withCredentials([file(credentialsId: 'flux-credentials', variable: 'FLUX_CREDENTIALS_FILE')]) {
                        sh """
                            flux apply -f .
                        """
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```text
{
  "commit": "abc123",
  "branch": "main",
  "pipeline": {
    "stage": "Deploy",
    "result": "success",
    "details": {
      "strategy": "canary-release",
      "percentage": "5%",
      "users": ["user1@example.com", "user2@example.com"]
    }
  },
  "environment": "prod",
  "status": "deployed"
}
```

**Data:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  source:
    repoURL: https://github.com/example/repo.git
    path: manifests
  destination:
    server: https://k8s.example.com
    namespace: default
```

**Flow:**

1. Yes
2. No
3. CI/CD Pipeline
4. Build
5. Test
6. Deploy
7. Is the feature ready?
8. Finalize deployment to allusers
9. Split test or rollback

