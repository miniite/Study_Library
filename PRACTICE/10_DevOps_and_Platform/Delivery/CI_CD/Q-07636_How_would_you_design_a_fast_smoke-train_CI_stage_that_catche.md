---
type: "practice"
difficulty: "Intermediate"
---

# How would you design a fast 'smoke-train' CI stage that catches obvious training breakage in minutes without waiting for a full run?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Emergency Deploy Paths](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Emergency_Deploy_Paths.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of designing a fast 'smoke-train' CI stage, we need to focus on creating a lightweight and efficient pipeline that can quickly validate critical parts of our training process without running an entire model training job. The goal is to catch obvious issues early in minutes rather than waiting for hours or days.

First, let's triage the question:

What are you probing? The interviewer is looking for your understanding of CI/CD principles and how to optimize them.

#### Assumptions
We assume the training pipeline involves multiple stages such as data preprocessing, model building, validation, and deployment. The 'smoke-train' stage should focus on these critical components.

Now, let's reason through the solution step by step from first principles:

Identify Critical Components:

Key steps in a typical ML training pipeline include data loading, initial model setup, basic preprocessing checks, a few epochs of training, and validation metrics.

Design the Pipeline:

Create a dedicated 'smoke-train' stage that runs only these critical components. This could be done by modifying an existing CI/CD pipeline or creating a new one.

Optimize Each Component:

Data Loading: Quickly fetch and validate a small subset of data.
Preprocessing: Run minimal preprocessing steps (e.g., normalization, scaling) to ensure the data is in the correct format.
Model Setup: Load pre-trained or simplified models for quick initialization.
Training: Perform just a few epochs with a reduced dataset. This can be done using an optimizer that stops early if no significant improvement occurs.
Validation Metrics: Compute key performance indicators (KPIs) like loss, accuracy, and F1 score to identify any immediate issues.

Automate and Integrate:

Use a CI/CD toolchain such as Jenkins, GitLab CI, or GitHub Actions to automate the 'smoke-train' stage.
Configure triggers (e.g., on commit) to run this pipeline frequently.
Set up notifications for any failures via Slack, email, etc.

Monitor and Improve:

Continuously monitor the performance of the 'smoke-train' stage.
Gradually increase the complexity based on feedback from the team.

#### Closing Summary
A well-designed 'smoke-train' CI stage will help catch obvious training breakages early by focusing on critical components. It requires careful selection and optimization of each step to ensure quick execution without compromising on accuracy or correctness.

#### Common Mistakes/Follow-Ups

Failing to identify and prioritize the correct steps for the 'smoke-train'.
Over-engineering the pipeline, leading to unnecessary complexity.
Not setting up proper monitoring and feedback mechanisms.

**Comparison:**

```
Stage	Component	Action
Smoke	Data Loading	Load small subset
	Preprocessing	Minimal steps
	Model Setup	Simplified model
	Training	Few epochs
	Validation	Key metrics
```

**Code:**

```python
# Example of a simplified smoke test in Python using a CI/CD tool like GitLab CI

stages:
  - smoke-test

smoke-test:
  script:
    - python3 data_loader.py --subset=small # Load subset of data
    - python3 preprocessor.py --preproc=light # Apply minimal preprocessing
    - python3 model_setup.py --model=simplified # Initialize a simple model
    - python3 train.py --epochs=2 --batch-size=16 # Train for 2 epochs with small batch size
    - python3 validator.py --metrics=loss,accuracy # Validate key metrics
```

**Example input / output:**

```text
Input:
- Commit a change in the training codebase

Output:
- Pipeline runs 'smoke-test' stage and identifies any obvious issues quickly, e.g.,:
  - Data preprocessing fails due to missing columns.
  - Model setup uses an incompatible architecture.
  - Training does not converge as expected.
```

**Data:**

```json
{
  "data_loader": {
    "subset": "small",
    "path": "/data/small_dataset.csv"
  },
  "preprocessor": {
    "preproc": "light",
    "columns": ["age", "gender"]
  },
  "model_setup": {
    "model": "simplified",
    "architecture": "SimpleFFNN"
  },
  "train": {
    "epochs": 2,
    "batch_size": 16
  },
  "validator": {
    "metrics": ["loss", "accuracy"]
  }
}
```

**Flow:**

1. Trigger
2. Data Loading
3. Preprocessing
4. Model Setup
5. Training for 2 Epochs
6. Validation Metrics
7. Notify Failure or Pass

