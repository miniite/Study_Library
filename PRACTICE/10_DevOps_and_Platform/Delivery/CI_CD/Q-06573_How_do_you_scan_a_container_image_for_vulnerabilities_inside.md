---
type: "practice"
difficulty: "Deep"
---

# How do you scan a container image for vulnerabilities inside a pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Container Images and Registries](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Container_Images_and_Registries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to scan a container image for vulnerabilities within a CI/CD pipeline, we'll break down the process from first principles and explore each step in detail. We'll cover the key components of setting up such a system, the trade-offs involved, and best practices.

In a CI/CD pipeline, scanning container images for vulnerabilities is crucial to ensure security. This involves integrating a vulnerability scanner into your build processes so that any issues are detected early in the development cycle. The primary goal is to automatically analyze container images and flag potential security threats before deployment.

The question is probing how you integrate a robust vulnerability assessment tool within your pipeline, along with understanding the nuances of different scanning tools, their capabilities, and the trade-offs involved. We need to make assumptions about the existing infrastructure (e.g., Docker, Kubernetes) and tools available for container management.

To start, we'll walk through the steps from fetching the source code to deploying the application securely:

Source Code Check-in: Developers commit changes to version control.
Build Phase: The CI server builds the Docker image(s).
Vulnerability Scan: The pipeline runs a vulnerability scanner on the built images.
Result Analysis and Notification: The results are analyzed, and notifications (e.g., Slack messages) are sent if issues are found.
Deployment Decision: Based on the scan results, the deployment decision is made.

Now let's dive into each step in detail:

Step 1: Source Code Check-in

Developers commit changes to a version control system like Git. A typical CI/CD pipeline would be triggered by this event.

Step 2: Build Phase

The CI server uses Docker or another containerization tool to build the image from the source code and configuration files.

Step 3: Vulnerability Scan

Here, we need to integrate a vulnerability scanner into our pipeline. Common tools include:

Clair: An open-source security advisory engine for container images.
Trivy: A lightweight command-line tool that integrates with Docker commands and supports multiple vulnerabilities databases.
OSS Index: A service by Sonatype that scans the packages used in the application.

Using Trivy as an example, we can integrate it into our pipeline via a Jenkinsfile or similar configuration. Below is a simplified example of how this might look:

GROOVY
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t my-app:latest .'
                }
            }
        }

        stage('Scan for Vulnerabilities') {
            steps {
                script {
                    // Scan the built image
                    sh 'trivy image --exit-code 1 my-app:latest'
                }
            }
        }
    }
}
Step 4: Result Analysis and Notification

After running the scan, we need to analyze the results. If vulnerabilities are found, we send notifications. This can be done through various integration points such as Slack, email, or a custom dashboard.

GROOVY
post {
    failure {
        // Send an alert via Slack
        script {
            emailext (
                subject: "Container Image Vulnerability Alert",
                to: 'security-team@example.com',
                body: "${env.BUILD_URL} contains security vulnerabilities."
            )
        }
    }
}
Step 5: Deployment Decision

Based on the scan results, the deployment process can be gated. If no critical issues are found, the application is deployed; otherwise, a human review or manual intervention might be required.

#### Closing Summary
and Common Mistakes/Follow-Ups

In conclusion, integrating vulnerability scanning into your CI/CD pipeline ensures that security is a continuous concern throughout the development lifecycle. Key points include choosing the right tool, configuring it correctly, handling false positives, and ensuring the process doesn't slow down the deployment.

#### Common mistakes
to avoid:

Overlooking regular updates: Vulnerability databases need to be updated regularly.
Not addressing high-severity vulnerabilities: Sometimes low-severity issues can be ignored too easily, leading to bigger problems later on.
False positives: Automated tools can sometimes flag non-vulnerable or outdated dependencies as security risks.

For follow-ups, we might discuss scaling the solution across multiple environments (e.g., different staging and production pipelines) or integrating with more complex DevOps practices like blue-green deployments.

**Comparison:**

```
Vulnerability Scanner	Key Features	Example Command
Clari	Integration with Kubernetes	clair scan --image my-app:latest
Trivy	Lightweight, CLI-based	`trivy image --exit-code 1 ...
OSS Index	Package analysis	osx index query "openssl" ...
```

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
                    // Build Docker image
                    sh 'docker build -t my-app:latest .'
                }
            }
        }

        stage('Scan for Vulnerabilities') {
            steps {
                script {
                    // Scan the built image
                    sh 'trivy image --exit-code 1 my-app:latest'
                }
            }
        }
    }

    post {
        failure {
            // Send an alert via Slack
            script {
                emailext (
                    subject: "Container Image Vulnerability Alert",
                    to: 'security-team@example.com',
                    body: "${env.BUILD_URL} contains security vulnerabilities."
                )
            }
        }
    }
}
```

**Example input / output:**

```text
Input:
- Source code changes in Git repository
- Docker image built from the source

Output:
- Vulnerability scan results for the Docker image
- Notification sent if high-severity vulnerabilities are found
- Image deployed or not based on the scan results
```

**Data:**

```json
{
  "trivyScanResults": [
    {
      "imageId": "sha256:a3b4c7d8e9f0",
      "vulnerabilities": [
        {
          "id": "CVE-2019-12345",
          "severity": "HIGH",
          "packageName": "openssl",
          "version": "1.1.1a"
        }
      ]
    }
  ]
}
```

**Flow:**

1. No High-Vulnerabilities
2. High-Vulnerabilities
3. Source Code Check-in
4. Build Docker Image
5. Vulnerability Scan with Trivy
6. Deploy Application
7. Send Alert via Slack

