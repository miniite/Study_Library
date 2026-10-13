---
type: "practice"
difficulty: "Intermediate"
---

# How do you deploy a static frontend to a CDN or object storage from a pipeline?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ CI CD Pipeline Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_CD_Pipeline_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're deploying a static frontend application to a Content Delivery Network (CDN) or Object Storage using a Continuous Integration/Continuous Deployment (CI/CD) pipeline. The key steps involve configuring build and deployment scripts, setting up environment variables, and ensuring the pipeline can handle both development and production environments.

The primary goal is to ensure that the frontend codebase is built and deployed efficiently without human intervention. This process typically involves several stages: building the application, testing it, and finally deploying the artifact to a CDN or object storage bucket.

#### Triage

We need to understand how the static frontend will be built and packaged, then deployed to a CDN or object storage. We'll assume we're using Git for version control, and our CI/CD pipeline is based on Jenkins or GitHub Actions (or any other CI/CD tool). The environment involves setting up secrets management, configuring build steps, and ensuring the deployment strategy is robust.

#### First-Principles Solution
Source Control: Use a Git repository to store the frontend code.
Build Step: Define a build step that compiles the static frontend (e.g., using Webpack).
Test Step: Run automated tests on the built artifact before deploying it.
Deployment: Use a deployment script or pipeline step to push the built files to a CDN or object storage bucket.
Detailed Steps

Git Repository Setup

Ensure the frontend code is stored in a Git repository, with versioned commits and branches (e.g., master for production, develop for development).

Build Configuration

Configure build tools like Webpack to compile the static assets.
Use a package manager like npm or Yarn to manage dependencies.

CI/CD Pipeline Setup

Define stages in the CI/CD pipeline:
Build Stage: Runs on every commit, compiles the frontend code.
Test Stage: Runs automated tests (e.g., unit tests, integration tests) on the built artifact.
Deploy Stage: Pushes the built files to a specified location.

Secrets Management

Use environment variables or secret management tools (like HashiCorp Vault) to store sensitive information such as API keys for the CDN or access tokens for object storage.

CDN/Object Storage Setup

Configure the target location in the pipeline configuration file.
Ensure the deployment script has the necessary permissions to write to the destination.

Deployment Script

Write a shell script or use a CI/CD tool's native commands to upload the built files to the CDN or object storage bucket.
Include error handling and logging to capture any issues during the deployment process.

#### Closing Summary

The CI/CD pipeline for deploying a static frontend involves setting up build steps, testing processes, and secure secrets management. The final step is configuring the deployment script to push the compiled files to a CDN or object storage bucket. This ensures that the application can be deployed reliably and efficiently without manual intervention.

#### Common Mistakes/Follow-Ups
Mistake: Forgetting to include automated tests in the pipeline.

#### Follow-Up
Discussing how to handle different environments (development, staging, production) and ensuring the deployment process is repeatable across all stages.

**Comparison:**

```
Stage	Action
Build	Run Webpack to compile frontend code.
Test	Execute unit tests and integration tests.
Deploy	Upload compiled files to CDN or S3 bucket.
```

**Code:**

```yaml
# Example Jenkinsfile snippet for a static frontend CI/CD pipeline

pipeline {
    agent any
    environment {
        CLOUDFRONT_BUCKET = 'example-cdn-bucket'
        S3_ACCESS_KEY_ID = credentials('s3-access-key-id')
        S3_SECRET_ACCESS_KEY = credentials('s3-secret-access-key')
    }
    stages {
        stage('Build') {
            steps {
                script {
                    // Run Webpack to compile the frontend code
                    sh 'npm run build'
                }
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                script {
                    withAWSAccessCredentials([id: 's3-credentials']) {
                        // Upload built files to S3
                        sh "aws s3 sync ./dist s3://${CLOUDFRONT_BUCKET}"
                    }
                }
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- Frontend codebase stored in Git repository (e.g., GitHub).
- CI/CD pipeline triggered by a push to the `master` branch.

Output:
- Compiled frontend assets uploaded to CDN or S3 bucket.
- Pipeline logs indicating successful build, test, and deployment stages.
```

**Data:**

```json
{
  "bucketName": "example-cdn-bucket",
  "accessKeyId": "YOUR_ACCESS_KEY_ID",
  "secretAccessKey": "YOUR_SECRET_ACCESS_KEY"
}
```

**Flow:**

1. Source Control
2. Git Repository
3. CI/CD Pipeline Triggers
4. Build Frontend
5. Test Frontend
6. Deploy to CDN or ObjectStorage

