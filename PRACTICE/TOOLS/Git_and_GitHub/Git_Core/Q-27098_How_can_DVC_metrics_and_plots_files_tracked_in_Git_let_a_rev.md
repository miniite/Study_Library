---
type: "practice"
difficulty: "Deep"
---

# How can DVC metrics and plots files, tracked in Git, let a reviewer see how a model's accuracy changed in a pull request?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Data with DVC](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Data_with_DVC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how DVC (Data Version Control) metrics and plots files can be used to track changes in a machine learning model’s accuracy within Git, let's break down the process from first principles.

In essence, reviewers need a way to assess whether the changes introduced by a pull request have affected the performance of the ML model. By integrating DVC with Git, we can ensure that metrics and plots are versioned alongside the code, making it easy for reviewers to see if the new commit has impacted model accuracy.

#### Triage
What is being Probed

The question here revolves around leveraging DVC to track machine learning model performance over time, specifically focusing on how a pull request can impact this metric. The key aspects include:

Understanding of Git and DVC workflows.
Ability to track and visualize changes in metrics.
Ensuring that tracking mechanisms are integrated into the development process.
Step-by-Step Solution

Integrate DVC with Git:

Initialize a new DVC project if it's not already set up.
Configure DVC to include relevant files like training logs, model parameters, and performance metrics (e.g., accuracy).

Define Metrics:

Identify the key metrics that need tracking for the ML model. For example, validation accuracy, F1 score, etc.

Track Model Performance:

After each training run, use DVC to stage changes in these metrics.
SH
dvc add <path_to_metrics_file>

Automate Metrics Collection:

Write a script that automatically collects and updates the metrics file after each training session.

Generate Plots and Dashboards:

Use DVC plots to generate visual representations of model performance.
SH
dvc plot --from <branch> --to <target_branch>

Commit Changes:

Commit both the metrics file and any changes in plots back to Git.

Review Pull Requests:

When a pull request is made, the reviewer can see the recent history of model performance directly from the DVC plots.
SH
dvc plot --from <base_branch> --to <pull_request_head>

Interpret Results:

The reviewer can compare the metrics and plots before and after the changes, to assess if there have been any adverse effects on model performance.

#### Closing Summary
and Common Mistakes

The integration of DVC with Git for tracking machine learning model performance ensures that all relevant data is versioned and easily accessible. This approach provides a clear historical record of how different commits affect the model’s accuracy, making it easier to review pull requests effectively.

#### Common Mistakes

Failing to initialize DVC properly.
Not automating metric collection, leading to manual errors.
Ignoring plot generation and only focusing on raw metrics.

#### Example
Workflow
SH
# Initialize a new DVC project
dvc init

# Add metrics file
dvc add model_metrics.json

# Train the model and update metrics
python train_model.py
# Manually or via script, update model_metrics.json with latest results

# Commit changes
git commit -m "Updated model metrics"

# Generate plots for review
dvc plot --from main --to feature_branch

