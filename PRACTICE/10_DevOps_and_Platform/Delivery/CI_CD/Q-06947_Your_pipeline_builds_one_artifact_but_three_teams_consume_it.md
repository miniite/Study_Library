---
type: "practice"
difficulty: "Intermediate"
---

# Your pipeline builds one artifact but three teams consume it with different feature flag defaults; how do you keep the artifact truly immutable while letting per-environment config diverge safely?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to ensure that a single build artifact is immutable across all teams and environments, but allow flexible configuration settings specific to each environment. This requires a robust CI/CD pipeline design that integrates versioning, separate build stages, and isolated deployment strategies.

At a high level, the goal is to have a unified artifact built in a reproducible manner from source control, with configurable defaults that can be overridden per-environment. The key steps involve setting up the pipeline, managing feature flags, and ensuring proper environment isolation.

First, we'll set up the CI/CD pipeline to build the artifact once and push it to a shared repository. This step ensures that all environments pull from the same immutable source. We’ll use GitLab, Jenkins, or another CI tool, which provides versioning and artifact storage features.

Next, for each environment (e.g., development, staging, production), we'll create separate build stages in our pipeline where different default configurations can be applied. This separation ensures that changes to one environment do not affect others while still allowing flexible settings.

Here's a step-by-step solution:

Source Control and Build Artifact Storage:

Use Git for version control.
Store the artifact in a registry like Docker Hub, Nexus, or AWS ECR.

Pipeline Configuration:

Define a pipeline that builds once from source code but splits into separate stages per environment.
Each stage can apply different feature flags and configuration settings specific to that environment.

Feature Flags Management:

Implement feature flags at the application level (e.g., using libraries like com.github.benmanes.caffeine.cache or custom flags).
Define a mechanism for toggling these flags based on environment metadata, which is set in pipeline configuration but not stored with the artifact.

Environment-Specific Configurations:

Use environment variables to manage differences between environments (development, staging, production).
For example, you could use Jenkins environment variables or Kubernetes secrets to inject specific configurations per environment.

Isolated Deployments:

Deploy each stage’s artifact to its respective environment using a separate CI/CD job.
Ensure no changes are pushed directly to the shared artifact repository; all modifications should go through the pipeline.

Let's dive into an example using Jenkins as our CI tool:

GROOVY
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def version = env.VERSION // Assume this is set from source control
                    docker.build("your-team-name/${env.BUILD_ID}:${version}")
                }
            }
        }
        stage('Development Build') {
            when {
                environment name: 'ENV', value: 'dev'
            }
            steps {
                script {
                    def artifact = docker.image("your-team-name/${env.BUILD_ID}:${env.VERSION}")
                    artifact.inside { // Run commands inside the container
                        sh 'echo "Dev Feature Flags: true"'
                        // Apply development-specific configurations here
                    }
                }
            }
        }
        stage('Staging Build') {
            when {
                environment name: 'ENV', value: 'staging'
            }
            steps {
                script {
                    def artifact = docker.image("your-team-name/${env.BUILD_ID}:${env.VERSION}")
                    artifact.inside { // Run commands inside the container
                        sh 'echo "Staging Feature Flags: beta"'
                        // Apply staging-specific configurations here
                    }
                }
            }
        }
        stage('Production Build') {
            when {
                environment name: 'ENV', value: 'prod'
            }
            steps {
                script {
                    def artifact = docker.image("your-team-name/${env.BUILD_ID}:${env.VERSION}")
                    artifact.inside { // Run commands inside the container
                        sh 'echo "Prod Feature Flags: off"'
                        // Apply production-specific configurations here
                    }
                }
            }
        }
    }
}

This example demonstrates how different stages can be configured to apply specific feature flags and environments, ensuring immutability of the artifact while allowing per-environment configuration.

**Code:**

```
GROOVY
Copy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                script {
                    def version = env.VERSION // Assume this is set from source control
                    docker.build("your-team-name/${env.BUILD_ID}:${version}")
                }
            }
        }
        stage('Development Build') {
            when {
                environment name: 'ENV', value: 'dev'
            }
            steps {
                script {
                    def artifact = docker.image("your-team-name/${env.BUILD_ID}:${env.VERSION}")
                    artifact.inside { // Run commands inside the container
                        sh 'echo "Dev Feature Flags: true"'
                        // Apply development-specific configurations here
                    }
                }
            }
        }
        stage('Staging Build') {
            when {
                environment name: 'ENV', value: 'staging'
            }
            steps {
                script {
                    def artifact = docker.image("your-team-name/${env.BUILD_ID}:${env.VERSION}")
                    artifact.inside { // Run commands inside the container
                        sh 'echo "Staging Feature Flags: beta"'
                        // Apply staging-specific configurations here
                    }
                }
            }
        }
        stage('Production Build') {
            when {
                environment name: 'ENV', value: 'prod'
            }
            steps {
                script {
                    def artifact = docker.image("your-team-name/${env.BUILD_ID}:${env.VERSION}")
                    artifact.inside { // Run commands inside the container
                        sh 'echo "Prod Feature Flags: off"'
                        // Apply production-specific configurations here
                    }
                }
            }
        }
    }
}
```

**Flow:**

1. Yes
2. No
3. Source Control
4. Build Artifact Storage
5. Is Build Artifacts Immutable?
6. Pipeline Stages forEnvironments
7. Development EnvironmentConfigurations
8. Staging EnvironmentConfigurations
9. Production EnvironmentConfigurations
10. Rebuild Artifacts
11. Environment-Specific FeatureFlags
12. Environment-Specific FeatureFlags
13. Environment-Specific FeatureFlags

