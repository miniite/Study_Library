---
type: "practice"
difficulty: "Expert"
---

# How would you attach provenance/attestation to a model artifact so you can prove which data, code, and run produced it during an audit?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build and Release Metadata](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure provable integrity and traceability of machine learning models in CI/CD pipelines, we need to attach metadata that captures the provenance of a model artifact, including details about the data, code, and run environment. This is crucial for audits and compliance checks. The key idea here is to design a system that logs and stores this information as part of the build process.

#### Triage
The question asks how to attach provenance/attestation to a model artifact in a CI/CD pipeline. Essentially, we need to ensure that each model artifact generated during development can be traced back to its source components and environments. The interviewer is probing our understanding of logging mechanisms, metadata handling, and integration with CI/CD tools.

#### Assumptions

We are using a robust CI/CD system like Jenkins or GitHub Actions.
Model artifacts are stored in a version-controlled repository (e.g., S3, GitLab).
We have access to the necessary environment details during model training/deployment.

#### First-principles solution

Metadata Logging: Define a structured metadata schema that captures all relevant information about the artifact generation process. This includes data source details, code commit hashes, environment configurations (e.g., hardware, OS), and any other pertinent parameters.
Automated Pipeline Integration: Integrate this logging mechanism into our CI/CD pipeline so that every time a model is built, the metadata is automatically collected and stored alongside the artifact.
Storage and Retrieval: Ensure the metadata is stored in a secure but accessible manner, such as an AWS S3 bucket or a database. This allows for easy retrieval during audits.

Let’s walk through a concrete example:

Suppose we are using a simple CI/CD pipeline with GitHub Actions. During each build step, we will log necessary details about the data and code versions used to train the model.

YAML
# Example GitHub Action workflow
name: Model Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  model_build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Train model
        env:
          DATA_PATH: ${{ secrets.DATA_PATH }}
          CODE_COMMIT_HASH: ${{ github.sha }}
        run: |
          python train.py

      - name: Log metadata
        run: |
          export ENVIRONMENT_DETAILS=$(echo "Hardware: GPU, OS: Ubuntu 20.04")
          echo "::set-output name=ENVIRONMENT_DETAILS::$ENVIRONMENT_DETAILS"

          echo "::set-output name=DATA_SOURCE::$(cat ${DATA_PATH})"
          echo "::set-output name=CODE_COMMIT_HASH::${CODE_COMMIT_HASH}"

      - name: Upload model artifact
        uses: actions/upload-artifact@v2
        with:
          name: model_artifact
          path: ./model.pkl

      - name: Save metadata to S3
        run: |
          aws s3 cp ${GITHUB_OUTPUT} s3://<bucket-name>/metadata/${CODE_COMMIT_HASH}.json

In this example, we use GitHub Actions to automate the collection of metadata. We capture details like DATA_PATH, CODE_COMMIT_HASH, and environment configurations. The metadata is then stored in an S3 bucket named <bucket-name>.

#### Closing summary
By attaching detailed metadata to model artifacts during the CI/CD process and storing it securely, we can provide robust provenance for our models. This approach ensures that every time a model is built or deployed, its lineage can be traced back to specific data sources, code versions, and environments used.

#### Common mistakes/follow-ups

Forgetting to automate metadata collection: Ensure the logging happens as part of the build pipeline.
Inadequate metadata schema: Include all relevant details (e.g., hardware, OS) in the metadata for comprehensive traceability.
Poor storage/retrieval mechanisms: Choose a reliable and accessible method for storing metadata (e.g., S3, database).

**Comparison:**

```
Metadata Field	Description
DATA_PATH	Path to the training data
CODE_COMMIT_HASH	Commit hash of the code used for model training
ENVIRONMENT_DETAILS	Details about the environment (e.g., hardware, OS)
```

**Code:**

```yaml
# Example GitHub Action workflow
name: Model Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  model_build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Setup Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.8'

      - name: Install dependencies
        run: |
          pip install -r requirements.txt

      - name: Train model
        env:
          DATA_PATH: ${{ secrets.DATA_PATH }}
          CODE_COMMIT_HASH: ${{ github.sha }}
        run: |
          python train.py

      - name: Log metadata
        run: |
          export ENVIRONMENT_DETAILS=$(echo "Hardware: GPU, OS: Ubuntu 20.04")
          echo "::set-output name=ENVIRONMENT_DETAILS::$ENVIRONMENT_DETAILS"

          echo "::set-output name=DATA_SOURCE::$(cat ${DATA_PATH})"
          echo "::set-output name=CODE_COMMIT_HASH::${CODE_COMMIT_HASH}"

      - name: Upload model artifact
        uses: actions/upload-artifact@v2
        with:
          name: model_artifact
          path: ./model.pkl

      - name: Save metadata to S3
        run: |
          aws s3 cp ${GITHUB_OUTPUT} s3://<bucket-name>/metadata/${CODE_COMMIT_HASH}.json
```

**Example input / output:**

```text
# Example input data for model training

echo "DATA_PATH=/path/to/data" > ${GITHUB_OUTPUT}
echo "CODE_COMMIT_HASH=$(git rev-parse HEAD)" >> ${GITHUB_OUTPUT}

# Metadata captured during the build step
export DATA_SOURCE="/path/to/data"
export CODE_COMMIT_HASH="abcdef1234567890"

# Environment details
export ENVIRONMENT_DETAILS='{"Hardware": "GPU", "OS": "Ubuntu 20.04"}'

echo "::set-output name=ENVIRONMENT_DETAILS::$ENVIRONMENT_DETAILS"
```

**Data:**

```json
{
  "DATA_PATH": "/path/to/data",
  "CODE_COMMIT_HASH": "abcdef1234567890",
  "ENVIRONMENT_DETAILS": {
    "Hardware": "GPU",
    "OS": "Ubuntu 20.04"
  }
}
```

**Flow:**

1. Start
2. Checkout repository
3. Setup Python
4. Install dependencies
5. Train model
6. Log metadata
7. Upload model artifact to S3
8. Save metadata to S3
9. End

