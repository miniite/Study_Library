---
type: "practice"
difficulty: "Deep"
---

# How would you keep build artifacts reproducible so the image you tested is exactly what ships?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Securing ML Delivery](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Securing_ML_Delivery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the build artifacts used during testing are exactly the same as those shipped, we need to focus on version control and reproducibility at both the code and artifact levels. This involves ensuring that every build is deterministic, using consistent environment configurations, and storing and managing builds in a controlled manner.

The question is essentially asking how to maintain a reliable and traceable build process from source to deployment. The interviewer might be probing your understanding of CI/CD pipelines, version control systems, and the importance of reproducibility in software development. Key assumptions include that you have access to common CI/CD tools like Jenkins or GitLab CI, and that the environment configurations are well-defined.

Let's break this down step by step:

Source Control and Versioning: Ensure all source code is under version control using a tool like Git. Each commit should be uniquely identifiable with a hash value.
Build Automation: Use a CI/CD pipeline to automate the build process, ensuring that every time you run it, the output is identical given the same input.
Environment Consistency: The build environment must be consistent and repeatable. This includes using the same versions of libraries, tools, and dependencies.
Artifacts Management: Store and version artifacts in a central repository like Docker Hub or AWS ECR to ensure they are accessible and trackable.

First-principles reasoning leads us through these steps:

Source Code Versioning:

Use Git for source code management.
Tag each commit with meaningful messages.
Use semantic versioning (e.g., v1.2.3) to denote changes.

Build Automation:

Define a CI/CD pipeline that runs on every push or pull request.
Use Dockerfiles to create reproducible build environments.
Ensure all dependencies are pinned in package managers like pip, npm, or maven.

Environment Consistency:

Use containerization (Docker) for consistent builds across different machines.
Define a .dockerignore file to exclude unnecessary files from the Docker image.
Use environment variables to manage configuration settings.

Artifact Management:

Store build artifacts in a central registry like AWS ECR, Docker Hub, or GitHub Packages.
Tag and version each artifact (e.g., v1.0.2).
Implement a policy for archiving old versions if necessary.

#### Closing summary
By implementing these practices, you ensure that the exact same build artifacts used during testing are what ship to production. Common mistakes include ignoring source control, not using environment variables, or failing to version and store artifacts properly.

**Comparison:**

```
Step	Action
1	Commit code changes to Git with a meaningful message.
2	Run CI/CD pipeline on push.
3	Build Docker image using the updated source code.
4	Tag and push Docker image to registry.
```

**Code:**

```bash
# Example Dockerfile snippet
FROM python:3.8-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["python", "main.py"]
```

**Example input / output:**

```text
Input:
- Source code changes pushed to Git repository.
- CI/CD pipeline triggered on push.

Output:
- Docker image built and tagged `v1.2.3` in ECR.
- Image pushed to ECR registry.
```

**Data:**

```
Docker Hub Repository:
JSON
Copy
{
  "name": "example-app",
  "tags": [
    "latest",
    "1.0"
  ]
}
```

**Flow:**

1. Source Code Changes
2. Commit to Git
3. Ci/CD Pipeline Triggers
4. Docker Image Built
5. Tagger & Push to Registry

