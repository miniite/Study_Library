---
type: "practice"
difficulty: "Basic"
---

# What happens to later steps in a job when an earlier step fails, by default?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pre-merge vs Post-merge Failures](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an earlier step in a CI/CD pipeline fails, the behavior of subsequent steps can vary based on how the pipeline is configured. By default, many CI/CD tools will stop executing further steps if one of them fails. This ensures that you don't spend time or resources on tasks that depend on a failed prerequisite.

However, it's important to understand that the default settings for failure behavior are highly configurable and can be customized according to the needs of your project. Let’s break this down from first principles:

Understanding CI/CD Pipelines: A CI/CD pipeline is composed of several steps or stages, each designed to perform a specific task in the software development lifecycle. These tasks could include building code, running tests, deploying applications, and more.

Default Behavior on Step Failure: By default, tools like Jenkins, GitLab CI, CircleCI, etc., are configured such that if an earlier step fails, subsequent steps will not be executed. This is because the failure of a prerequisite task may indicate an issue in the codebase or configuration that needs to be addressed before proceeding.

Customizing Failure Behavior: While the default behavior stops execution on failure, you can configure your pipeline to continue executing subsequent steps despite earlier failures. For example, you might want to log errors and proceed with other tasks to avoid wasting time on tests that depend on the failed step. This can be particularly useful in multi-stage pipelines where some steps are independent of each other.

Why Default Behavior Makes Sense: Stopping execution on failure helps prevent the propagation of faulty code or configurations into later stages, which could lead to cascading errors and wasted resources. It is a robust approach to ensure that your system remains stable and reliable.

#### Common Mistakes
and Considerations:

Failing to understand the default behavior can lead to overlooking critical issues if you rely on subsequent steps.
Overriding this behavior without careful consideration may result in incomplete or broken pipelines, leading to potential security vulnerabilities or performance issues.

#### Closing summary
The default behavior of CI/CD pipelines is generally to stop executing further steps when an earlier step fails. This approach helps maintain the integrity and reliability of your pipeline but can be customized based on specific project needs.

**Code:**

```yaml
# Example Jenkinsfile snippet
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }
        stage('Test') {
            when { not failed() } // Only run if previous stage was successful
            steps {
                echo 'Running tests...'
            }
        }
        stage('Deploy') {
            when { not failed() } // Only run if previous stages were successful
            steps {
                echo 'Deploying application...'
            }
        }
    }
}
```

**Flow:**

1. Build
2. Test
3. Deploy
4. Post-Deploy Checks

