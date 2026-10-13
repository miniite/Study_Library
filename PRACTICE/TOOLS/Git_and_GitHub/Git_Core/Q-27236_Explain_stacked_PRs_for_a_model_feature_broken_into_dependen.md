---
type: "practice"
difficulty: "Deep"
---

# Explain stacked PRs for a model feature broken into dependent layers (data prep, training change, serving change): how they map to Git branches, and the exact pain when a lower PR is squash-merged, and how a tool like Graphite handles it.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Stacked pull requests (PRs) are a common workflow in software development, especially when working on complex projects with multiple interdependent components. In the context of an ML model feature broken into dependent layers such as data preparation, training change, and serving change, stacked PRs can be visualized using Git branches. This approach helps maintain a clean and understandable history of changes by separating each layer's development process.

#### Triaging the Question

The question is probing how we manage dependencies between different parts of an ML model across multiple PRs and branches in Git/GitHub. Specifically, it asks about the pain points that arise when a lower-level PR (e.g., data preparation) is squash-merged into the main branch before its higher-level counterpart (e.g., training change or serving change), and how tools like Graphite handle such scenarios.

#### Assumptions
We are working with a complex ML model feature that requires changes across multiple layers: data preparation, training, and serving.
These changes are being managed using Git branches for better isolation and collaboration.
The goal is to manage dependencies between these layers without causing integration issues when merging.

#### First-Principles Solution

Let's break down the workflow step by step:

Branching Strategy:

Each layer of the model feature (data preparation, training change, serving change) will have its own Git branch.
These branches are created from a common base branch where all ongoing development is tracked.

Dependent PRs:

A pull request for data preparation will be opened and reviewed independently.
Once the data preparation changes pass code review, they will be merged into the main branch (or another staging branch) to ensure that the next layers can build on top of the updated data prep.

Conflict Management:

When a higher-level layer (e.g., training change) depends on the lower-level layer (data preparation), it should create its own PR based on the latest version of the data preparation branch.
This ensures that any changes in the lower level do not break the higher-level components during development.

Squash-Merge Considerations:

If a lower-level PR (e.g., data prep) is squash-merged into the main branch, it means all commits are squashed into one commit before merging.
This can cause issues if there are conflicting changes in other layers that were not merged until after this PR was squash-merged.
Pain Points with Squash-Merge
Integration Issues:
Higher-level layers (training or serving) may break due to the sudden change from a single commit, leading to integration problems.
Loss of Commit History:
Squash merging results in loss of individual commit history, making it harder to trace and debug issues if they arise.
How Graphite Handles It

Graphite is an open-source tool designed specifically for managing complex workflows involving interdependent PRs. Here’s how it addresses the pain points:

Dependency Management:

Graphite allows defining dependencies between PRs, ensuring that certain PRs cannot be merged until their prerequisites are in place.
This prevents squash-merging issues by ensuring all dependent layers are up-to-date before any changes are made.

Visual Workflow:

Graphite provides a visual interface to manage and track the flow of PRs, making it easier for team members to understand dependencies and ensure proper sequencing.
It helps in identifying bottlenecks and coordinating reviews more effectively.

Automated Conflict Resolution:

Graphite can automatically detect conflicts and surface them before merging, helping prevent accidental squash-merges that could cause issues.

#### Closing Summary

Stacked PRs for dependent layers of an ML model are managed using Git branches to ensure clear separation of concerns and dependency tracking. Squash-merging a lower-level PR into the main branch can introduce pain points such as integration issues and loss of commit history, which Graphite addresses by managing dependencies visually and preventing accidental squash merges.

#### Common Mistakes
and Follow-Ups
Common Mistake: Failing to create separate branches for each layer leads to mixed-up codebases that are harder to manage.

#### Follow-Up
Discussing the trade-offs between using a monorepo versus multi-repository approach in complex ML projects can provide additional insights.

**Comparison:**

```
Layer	Branch Name	Description
Data	feature/prepare_data	Prepares input data for model.
Train	feature/training_improvements	Updates training logic to use new data.
Serve	feature/serving_updates	Enhances serving layer with trained model.
```

**Code:**

```python
# Example of branching strategy
def create_branch(repo, branch_name):
    repo.create_head(branch_name)

# Example PR creation
def create_pr(repo, base_branch, head_branch, title, body):
    pr = repo.create_pull(title=title, body=body, base=base_branch, head=head_branch)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Create PR for `data_prep` and merge it into main. <br> Create PR for `training` based on latest `data_prep`. <br> Create PR for `serving` based on latest `training`. | Data prep changes -> Training updates -> Serving adjustments. Integration issues may arise if data prep is squash-merged before training is ready. |

**Data:**

```yaml
model_features:
  - data_prep: "feature/prepare_data"
  - training: "feature/training_improvements"
  - serving: "feature/serving_updates"
```

**Flow:**

1. Data Preparation PR
2. Create Data Prep Branch
3. Merge into Main
4. Training Change PR
5. Create Training Branch
6. Based on Latest Data Prep
7. Serving Change PR
8. Create Serving Branch
9. Based on Latest Training

