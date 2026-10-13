---
type: "practice"
difficulty: "Intermediate"
---

# How would you promote a single, immutable anomaly-model artifact through dev, staging, and prod so the exact model you evaluated is the one that scores live telemetry?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build Reproducibility and Artifact Promotion](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/Build_Reproducibility_and_Artifact_Promotion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that an evaluated anomaly detection model remains immutable and consistent across development (dev), staging, and production environments, we need to implement a robust CI/CD pipeline. This involves several key steps: versioning models, automating deployment, and ensuring that the exact model used for evaluation is deployed in production. We'll break this down from first principles.

#### Triaging the Question

The core of the question is about deploying an immutable anomaly detection model across different environments (dev, staging, prod). The interviewer is probing your understanding of CI/CD practices, version control strategies, and deployment automation.

#### Assumptions
and Key Ideas
Version Control: The model will be managed in a version-controlled environment.
Environment Isolation: Dev, staging, and production environments are isolated to prevent accidental changes during evaluation or deployment.
Automation: CI/CD pipelines automate the process of deploying models.

#### First-Principles Solution

Model Versioning:

Use a version control system like Git for model artifacts.
Each time you evaluate an anomaly detection model, tag it with a unique version identifier (e.g., v1, v2).

CI/CD Pipeline Setup:

Set up CI pipelines to run tests on the models in dev environments.
Use staging environments for final testing before going live.

Automated Deployment:

Create a CI/CD pipeline that automates the deployment of the exact model version from development to staging and then to production.
Ensure the deployment process is repeatable and verifiable.

Environment Configuration Management:

Use environment-specific configuration files or variables to manage differences between dev, staging, and prod environments.
Leverage tools like Kubernetes for managing deployments across different environments.

Monitoring and Logging:

Implement monitoring to track the performance of the deployed model in real-time.
Log all deployment activities for auditing and troubleshooting.

#### Closing Summary

By setting up a robust CI/CD pipeline, versioning models effectively, and automating the deployment process, we can ensure that the exact model evaluated in dev is the one used in production. This approach not only maintains consistency but also provides a clear audit trail and easy rollback mechanisms if issues arise.

#### Common Mistakes/Follow-Ups

Mistake: Not versioning models properly.

#### Follow-Up
Ensure you use a consistent tagging strategy for model versions.

Mistake: Failing to automate deployment.

#### Follow-Up
Use CI/CD tools like Jenkins, GitLab CI, or Kubernetes to automate the process.

**Comparison:**

```
Model Version	Date	Description
v1	2023-09-20	Initial release based on dataset V1
```

**Code:**

```bash
# Example of a basic CI/CD pipeline script in YAML (Jenkinsfile)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building model artifact'
                sh 'python train_model.py --output-model output/model_v1.pkl'
            }
        }

        stage('Test') {
            when { branch 'dev' }
            steps {
                echo 'Running tests in dev environment'
                sh 'pytest tests/dev_tests.py'
            }
        }

        stage('Deploy Staging') {
            when { branch 'staging' }
            steps {
                echo 'Deploying model to staging environment'
                sh 'kubectl apply -f deployment/staging-deployment.yaml'
            }
        }

        stage('Deploy Prod') {
            when { tag matches '^v.*$' } // Only deploy tagged versions
            steps {
                echo 'Deploying model to production environment'
                sh 'kubectl apply -f deployment/prod-deployment.yaml --tag=v1'
            }
        }
    }
}
```

**Example input / output:**

```text
# Example dry-run input for a deployment to staging

Input:
- Model version v1
- Tag applied in the pipeline

Output:
- Model artifact 'output/model_v1.pkl' deployed to staging environment with container image tag 'v1'
- Pod created: anomaly-model-staging-76f457c5b9-bzr8p
```

**Data:**

```yaml
# Example model version schema
model_versions:
  - id: v1
    date: 2023-09-20
    description: Initial release based on dataset V1

# Example deployment configuration for staging
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anomaly-model-staging
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: anomaly-model
    spec:
      containers:
      - name: anomaly-detection
        image: registry.example.com/anomaly-detection:v1
```

**Flow:**

1. Model Evaluation
2. Tag Model with Version
3. CI Pipeline - Build and Test inDev
4. Deploy Staging Environment
5. Final Testing & Approval
6. Tag Model for ProdDeployment
7. Kubernetes Deploy Prod
8. Monitor ProductionPerformance

