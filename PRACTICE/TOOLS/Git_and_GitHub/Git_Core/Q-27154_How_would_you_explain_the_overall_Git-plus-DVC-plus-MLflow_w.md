---
type: "practice"
difficulty: "Deep"
---

# How would you explain the overall Git-plus-DVC-plus-MLflow workflow you'd use to keep an ML project reproducible from data to deployed model?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Data with DVC](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure an ML project is reproducible, leveraging a combination of Git, DVC (Data Version Control), and MLflow allows us to manage the entire lifecycle of our models, from data preparation to deployment. Essentially, this workflow provides a structured approach to version control that spans across different aspects of the machine learning pipeline.

The core idea here is that each component serves a specific purpose: Git manages code changes, DVC handles data and model versioning, and MLflow tracks experiments and models. Together, they create a robust framework for maintaining reproducibility in an ML project.

First, we will break down what these tools do and then show how to integrate them effectively.

**Comparison:**

```
Git Commands	Description
git init	Initialize a new Git repository
git add .	Stage all changed files for the next commit
git commit -m "Commit message"	Create a new commit with the staged changes
DVC Commands	Description
dvc add <file>	Stage a file for version control
dvc repro	Reproduce all dependencies and outputs defined in dvc.yml
dvc push	Push data to a remote DVC repository (e.g., GitHub, GitLab)
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Git] -->|Track code changes| B[Local Project]
    C[DVC] -->|Manage data & model versions| B
    D[MLflow] -->|Log experiments & track models| B
```

**Example input / output:**

```

Here’s a simplified example of how you might use these tools in practice:

Initialize DVC and Git:

BASH
Copy
git init
dvc init

Stage data for versioning with DVC:

BASH
Copy
dvc add data/raw/train.csv

Track changes in code using Git:

BASH
Copy
git commit -m "Initial setup"

Train a model and log it using MLflow (assuming Python environment):

PYTHON
Copy
import mlflow

with mlflow.start_run(run_name="Model Training"):
    # Load data
    X, y = load_data("data/processed/train_processed.csv")

    # Train the model
    model = train_model(X, y)

    # Log parameters and metrics
    mlflow.log_param("learning_rate", 0.1)
    mlflow.log_metric("accuracy", 0.95)

    # Save the model to DVC
    dvc_add("model.pkl")
    mlflow.pyfunc.log_model(model_uri="dvc://model/params.yaml",
                            artifacts=["model.pkl"])

Commit changes:

BASH
Copy
git add . ; git commit -m "Logged model training run"
```

**Data:**

```

The project structure might look like:

YAML
Copy
.
├── .dvc  # DVC configuration directory
│   ├── dvc.yml
│   └── params.yaml
├── data  # Raw, processed, and versioned data
│   ├── raw
│   │   └── train.csv
│   └── processed
│       └── train_processed.csv
├── models  # Model versions
│   ├── model_version_1
│   └── model_version_2
└── src  # Source code for the ML project
    ├── data.py
    ├── model.py
    └── trainer.py
```

**Flow:**

1. Repository
2. Project
3. Git
4. Track code changes
5. DVC
6. Manage data & modelversions
7. MLflow
8. Log experiments & trackmodels
9. Staging area in Git
10. Data versioned with DVC
11. Models logged with MLflow

