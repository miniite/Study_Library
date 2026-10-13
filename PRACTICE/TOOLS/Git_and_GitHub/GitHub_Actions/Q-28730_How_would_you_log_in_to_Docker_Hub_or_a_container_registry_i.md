---
type: "practice"
difficulty: "Intermediate"
---

# How would you log in to Docker Hub or a container registry inside a workflow?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To log into Docker Hub or any container registry from within a GitHub Actions workflow, we need to securely manage credentials and ensure that our actions can authenticate properly. This process involves setting up secrets in GitHub, configuring the correct commands, and using best practices for handling sensitive information.

#### Triage

The question is essentially asking how to automate the login process for Docker Hub or any container registry within a GitHub Actions workflow. The key points are ensuring security, managing credentials, and executing the necessary authentication commands.

#### Assumptions
You have already set up a repository with GitHub Actions.
You need to push or pull images from a container registry like Docker Hub.
You want to keep your secrets secure but accessible by the actions in the workflow.

#### Solution

From first principles, we start by understanding that authenticating with a container registry requires credentials. These credentials are typically stored as encrypted secrets in GitHub and then used within the workflow.

Set Up Secrets:

Go to your repository settings on GitHub.
Navigate to "Secrets" under the "Settings" tab.
Add two secrets:
DOCKER_USERNAME: Your Docker Hub username or registry-specific credentials.
DOCKER_PASSWORD: The corresponding password/token for the user/registry.

Configure the Workflow:

Create a .github/workflows directory if it doesn't exist in your repository.
Inside this folder, create a YAML file that defines the workflow (e.g., docker-login.yml).

Write the Workflow File: Here’s an example of how to structure the workflow:

YAML
name: Docker Login

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  login:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker buildx (if needed)
        run: |
          # This is optional if your images don't require buildx
          docker buildx create --name multi-platform-build --use

      - name: Log in to Docker Hub
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
        run: |
          echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin

#### Closing Summary

By following these steps, you can securely log into Docker Hub or any container registry within a GitHub Actions workflow. The key is to store your credentials as secrets in GitHub and use the --password-stdin option with docker login to avoid hardcoding them directly into your workflow file.

#### Common Mistakes/Follow-Ups
Mistake 1: Hardcoding sensitive information like passwords directly in the workflow. Always use environment variables or secrets.
Mistake 2: Not using the --password-stdin option, which can expose credentials via logging or other unintended mechanisms.

#### Follow-Up
If you are working with multiple registries, consider creating a more dynamic approach to handle different login steps based on image tags or other metadata.

**Code:**

```yaml
name: Docker Login

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  login:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up Docker buildx (if needed)
        run: |
          # This is optional if your images don't require buildx
          docker buildx create --name multi-platform-build --use

      - name: Log in to Docker Hub
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
        run: |
          echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
```

