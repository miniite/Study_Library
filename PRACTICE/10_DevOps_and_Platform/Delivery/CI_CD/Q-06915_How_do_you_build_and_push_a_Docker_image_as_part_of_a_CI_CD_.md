---
type: "practice"
difficulty: "Intermediate"
---

# How do you build and push a Docker image as part of a CI/CD pipeline?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Container Images and Registries](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Container_Images_and_Registries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When building a CI/CD pipeline that involves pushing a Docker image, we need to ensure our processes are automated, reliable, and secure. At a high level, this involves setting up a series of steps where code changes trigger a build process, which then generates a Docker image. This image is subsequently tagged, tested (if necessary), and pushed to a container registry like Docker Hub or Amazon ECR.

To break down the solution from first principles:

#### Triage

The question asks how to integrate Docker image building into a CI/CD pipeline. The key aspects include:

Automation: Ensuring the process is automated.
Security: Safeguarding sensitive information such as API keys and credentials.
Testing: Optionally, running tests on the built Docker image before pushing it.
Version Control: Properly tagging images with meaningful version tags.

#### Assumptions
We are using Git for source control.
The pipeline is triggered by code changes (e.g., via a webhook).
We have access to a container registry like Docker Hub or Amazon ECR.
Step-by-Step Solution

Source Control and Webhook Setup:

Ensure your repository has webhooks configured to trigger builds when there are code changes.

Build Stage:

Use a CI tool (e.g., Jenkins, GitHub Actions, GitLab CI) to define the build process.

#### Example
using Dockerfiles: Define a Dockerfile in your project's root directory.
DOCKER
# Dockerfile
FROM node:14

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
Image Tagging and Pushing:
Use a script or CI step to tag the Docker image with a version number derived from Git tags or commit messages.
BASH
# Script for tagging and pushing an image (example using AWS ECR)
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

docker build -t my-app:$GIT_COMMIT_SHORT .
docker tag my-app:$GIT_COMMIT_SHORT <account-id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
Testing (Optional):
Run tests on the Docker image using tools like docker-compose or a custom test runner.
YAML
# docker-compose.yml for testing
version: '3'
services:
  my-service:
    build: .
    command: npm test

Promotion and Staging (Optional):

Deploy the Docker image to staging environments before pushing it to production.

Security Considerations:

Use environment variables or secure vaults for storing credentials.
Enable network policies and access controls in your container registry.

#### Closing Summary

In summary, building and pushing a Docker image as part of a CI/CD pipeline involves defining automated steps that cover code changes, build processes, tagging, testing (if needed), and pushing to a secure registry. This ensures a robust and maintainable deployment process.

#### Common Mistakes/Follow-Ups

Mistake: Failing to automate the entire pipeline.

#### Follow-Up
Ensure all steps from code changes to image deployment are automated.

Mistake: Not properly securing credentials.

#### Follow-Up
Use environment variables or secure vaults for storing sensitive information.

**Comparison:**

| Step | Action                  |
|------|------------------------|
| 1    | Set up webhooks        |
| 2    | Define Dockerfile      |
| 3    | Create build script    |
| 4    | Tag and push image     |
| 5    | Run tests              |
| 6    | Deploy to staging      |

**Code:**

```bash
# Example script for tagging and pushing an image
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

docker build -t my-app:$GIT_COMMIT_SHORT .
docker tag my-app:$GIT_COMMIT_SHORT <account-id>.dkr.ecr.<region>.amazonaws.com/my-app:latest
```

**Example input / output:**

```text
$ aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com

$ docker build -t my-app:123abc .
$ docker tag my-app:123abc 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:latest
```

**Data:**

```yaml
# Example docker-compose.yml for testing
version: '3'
services:
  my-service:
    build: .
    command: npm test
```

**Flow:**

1. Code Change
2. CICD Trigger
3. Build Docker Image
4. Tag and Push Image
5. Test Image (Optional)
6. Promote to Staging (Optional)
7. Deploy to Production

