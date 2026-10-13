---
type: "practice"
difficulty: "Basic"
---

# How would you structure commits when building a training pipeline so each stage (load, preprocess, train, evaluate) is a clear checkpoint?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When structuring commits in Git and GitHub for a machine learning project with multiple stages like loading data, preprocessing, training, and evaluating, it's crucial to ensure each commit clearly represents a discrete step or change. This not only makes the code history clean but also aids collaboration and reproducibility.

#### Triage

The question is essentially asking how to organize commits in Git so that each stage of the machine learning pipeline (load data, preprocess, train model, evaluate) has its own clear checkpoint. The interviewer might be probing your understanding of version control best practices and how to manage a complex project with multiple stages.

#### First-Principles Solution

To structure these commits effectively:

Commit Naming: Each commit message should clearly describe the change made in that commit.
Branching Strategy: Use branches for different stages or features.
Staging and Review: Ensure each stage is tested and reviewed before merging into a main branch.
Step-by-Step Solution
1. Branching for Stages

Start by creating separate branches for each stage of the pipeline:

BASH
git checkout -b load-data
git commit -m "Add data loading script"
git checkout -b preprocess
git commit -m "Add preprocessing logic"
git checkout -b train
git commit -m "Implement training logic"
git checkout -b evaluate
git commit -m "Add evaluation scripts"
2. Commit Naming and Message

Ensure each commit message is clear and descriptive:

Load Data: Add data loading script
Preprocess: Add preprocessing logic for data
Train Model: Implement training logic with hyperparameters X, Y, Z
Evaluate Model: Add evaluation scripts with metrics A, B, C
3. Testing and Review

Before merging each branch into the main branch (e.g., main), perform thorough testing:

BASH
git checkout main
git merge --no-commit preprocess  # Interactive rebase for conflict resolution
git merge --no-commit train
git merge --no-commit evaluate

Ensure all stages pass their respective tests before committing changes to main.

4. Commit Messages and Comments

Use comments within the code to explain key decisions or complex logic:

PYTHON
# This function handles data loading and preprocessing.
def load_and_preprocess_data():
    # Load raw data from CSV.
    data = pd.read_csv('raw_data.csv')

    # Preprocess data by cleaning, handling missing values, etc.
    processed_data = clean_data(data)
    return processed_data

# Hyperparameters are chosen based on initial trials.
params = {
    'learning_rate': 0.01,
    'batch_size': 32
}

#### Closing Summary
and Common Mistakes/Follow-Ups

By structuring your commits in this way, you ensure that each stage of the pipeline is clearly documented and testable. Key mistakes to avoid include:

Ignoring Intermediate Stages: Each step should have its own commit.
Overly Complicated Commit Messages: Keep messages concise but descriptive.

For follow-ups, consider discussing how to handle dependencies between stages or how to integrate version control with continuous integration tools like GitHub Actions.

**Code:**

```bash
git checkout -b load-data
git commit -m "Add data loading script"
git checkout -b preprocess
git commit -m "Add preprocessing logic"
git checkout -b train
git commit -m "Implement training logic"
git checkout -b evaluate
git commit -m "Add evaluation scripts"
```

**Example input / output:**

```text
# Example workflow
$ git checkout main
$ git merge --no-commit preprocess  # Resolve conflicts interactively
$ git merge --no-commit train
$ git merge --no-commit evaluate
```

