---
type: "practice"
difficulty: "Deep"
---

# How would you version a feature transformation so the pipeline can guarantee the exact same code path runs offline (training) and online (serving)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Changes in Delivery Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Schema_Changes_in_Delivery_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To version a feature transformation in a CI/CD pipeline such that both training and serving use the exact same code path, we need to ensure that any changes made during development are properly captured and deployed without causing discrepancies between offline (training) and online (serving) environments. This involves setting up clear processes for managing transformations and ensuring reproducibility at every stage.

#### Triaging the Question

The question is asking how to version control feature transformations in a way that guarantees consistency across different environments—specifically, offline training pipelines and online serving pipelines. The core of this problem lies in maintaining a single source of truth for the code while managing its lifecycle effectively. Key assumptions include:

The transformation logic can be represented as code.
Both training and serving environments are using the same versioned code.

#### Solution
from First Principles
Code Versioning: Use a Git-based repository to manage your feature transformations. Each commit should represent a specific state of the transformation logic.
Environment-specific Tags: Apply tags or branches in the repository that correspond to different stages (e.g., dev, staging, prod).
CI/CD Pipeline Setup:
Stages: Define distinct CI/CD pipeline stages for each environment.
Automation: Automate the process of fetching and deploying specific code versions based on the current stage.
Reproducibility: Ensure that every run, whether training or serving, has a consistent starting point by using deterministic builds.
Detailed Solution

Repository Structure

PLAINTEXT
/feature_transforms/
    v1/
        transform.py
    v2/
        transform.py

Each directory represents a different version of the transformation logic.

CI/CD Pipeline Configuration:

Stages: Define stages such as build, test, and deploy.
Triggers: Set up triggers for each environment.
YAML
pipelines:
  - name: Training Pipeline
    branches:
      - main
    steps:
      - name: Build Training Environment
        image: python:3.9
        commands:
          - pip install -r requirements.txt
          - cp /feature_transforms/latest/transform.py .
      - name: Run Training Job
        image: training-env
        commands:
          - python train_model.py

  - name: Serving Pipeline
    branches:
      - main
    steps:
      - name: Build Serving Environment
        image: python:3.9
        commands:
          - pip install -r requirements.txt
          - cp /feature_transforms/stable/transform.py .
      - name: Deploy Model to Serving
        image: serving-env
        commands:
          - python serve_model.py

Deterministic Builds: Use Docker containers and container orchestration tools (e.g., Kubernetes) to ensure that the same environment is used for both training and serving.

YAML
dockerfile:
  FROM python:3.9-slim
  COPY . /app
  WORKDIR /app
  RUN pip install -r requirements.txt
  CMD ["python", "train_model.py"]

Tagging and Branching: Use Git tags or branches to manage different versions.

BASH
git tag v1.0
git checkout -b staging

Automated Testing: Ensure that every commit goes through automated testing before being deployed.

#### Closing Summary

By using a combination of versioned code, CI/CD pipelines with environment-specific stages, and deterministic builds, we can ensure that both training and serving environments use the exact same feature transformation logic. This approach guarantees reproducibility and consistency across different environments, which is crucial for maintaining model integrity in machine learning deployments.

#### Common Mistakes/Follow-Ups
Not Using Version Control: Failing to manage code changes through a version control system can lead to discrepancies.
Manual Deployment: Manual deployment processes are error-prone and can introduce inconsistencies.
Inconsistent Environments: Different environments with different versions of dependencies or configurations can cause issues.

**Comparison:**

| Stage    | Commands                    |
|----------|-----------------------------|
| Build Training Environment | cp /feature_transforms/latest/transform.py .  |
| Run Training Job         | python train_model.py        |

**Code:**

```yaml
# Example CI/CD Pipeline Configuration (Jenkinsfile)
pipeline {
    agent any
    stages {
        stage('Build Training Environment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'cp /feature_transforms/latest/transform.py .'
                    }
                }
            }
        }
        stage('Run Training Job') {
            steps {
                sh 'python train_model.py'
            }
        }
    }
}
```

**Example input / output:**

```text
Input (commit to main branch):
- Add new feature transformation logic in `v2/transform.py`
- Tag the version with `git tag v1.0`

Output:
- CI pipeline triggers, building training and serving environments using `v1.0` code.
- Deployed models use the exact same transformations across both pipelines.
```

**Data:**

```json
{
  "pipeline": {
    "stages": [
      { "name": "Build Training Environment", "commands": ["cp /feature_transforms/latest/transform.py ."] },
      { "name": "Run Training Job", "commands": ["python train_model.py"] }
    ]
  }
}
```

**Flow:**

1. Git Commit
2. CI Pipeline Trigger
3. Build Docker Image (Training)
4. Deploy Model to TrainingEnvironment
5. Git Commit
6. CI Pipeline Trigger
7. Build Docker Image (Serving)
8. Deploy Model to ServingEnvironment

