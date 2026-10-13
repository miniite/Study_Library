---
type: "practice"
difficulty: "Deep"
---

# You want to roll back a fine-tuned model to a prior adapter version if the new one regresses on evals. How would you tie the adapter artifact to a Git tag so the rollback is one deterministic step?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to integrate Git tags with our model artifacts in such a way that rolling back to a previous version can be done deterministically. The core idea here is to associate each adapter version with a unique Git tag and then use these tags to manage and revert the model's state when necessary.

#### Triage

The question essentially asks how to tie a specific version of an adapter (a component in our fine-tuned model) to a Git tag, allowing us to perform deterministic rollbacks if the new adapter version degrades performance on evaluation metrics. We need to ensure that:

Each adapter version is uniquely tagged.
Rolling back can be done by simply switching tags.

#### Assumptions
and First Principles
Git Workflow: The model repository uses Git for version control.
Adapter Artifacts: These are stored in a designated directory within the repository.
Evaluation Metrics: We have predefined evaluation metrics to determine if the new adapter regresses.

#### Solution
1. Tagging Adapter Versions

We will use Git tags to label each adapter version. Each tag will correspond to a specific commit that contains the artifact of interest (e.g., adapter-v2.3).

BASH
git tag -a adapter-v2.3 -m "Version 2.3 of the adapter"
2. Storing Adapter Artifacts

The artifacts will be stored in a directory within the repository, named after their corresponding tags.

PLAINTEXT
/model-artifacts/
    ├── adapter-v1.0.tar.gz
    ├── adapter-v2.0.tar.gz
    └── adapter-v2.3.tar.gz
3. Model Initialization and Evaluation

When initializing the model for training or inference, we will load the latest version from a default branch (e.g., main). For rollbacks, we can specify the tag directly.

PYTHON
def load_adapter(tag):
    artifact_path = f"/model-artifacts/adapter-{tag}.tar.gz"
    # Load and initialize the model using the artifact
4. Detecting Regression

Before deploying a new adapter version, we perform evaluation on predefined datasets to check for regression. If detected, we rollback by updating the tag used in our initialization code.

PYTHON
def evaluate_adapter(new_tag):
    old_tag = get_current_tag()  # Retrieve current active tag
    if is_regression(new_tag):  # Define this function based on your criteria (e.g., eval loss increase)
        rollback_to_tag(old_tag)  # Rollback to the previous version

def rollback_to_tag(tag):
    new_tag = tag
    load_adapter(new_tag)

# Example:
evaluate_adapter("adapter-v2.3")
5. Deployment and Rollback Automation

For continuous integration/continuous deployment (CI/CD) pipelines, we can automate this process using Git hooks or custom scripts that run during the CI steps.

#### Closing Summary

By associating each adapter version with a Git tag and storing these artifacts in corresponding directories, we ensure deterministic rollbacks. The key is to maintain clear documentation of which tags correspond to which versions and to update our model loading logic accordingly.

#### Common Mistakes/Follow-Ups
Mistake: Not tagging adapters properly or missing steps in the rollback process.

#### Follow-Up
How do you handle multiple changes across different components? Do we need a more sophisticated branching strategy?

#### Follow-Up
What if there are multiple regressions, and how do we manage rollbacks to intermediate states?

**Code:**

```bash
git tag -a adapter-v2.3 -m "Version 2.3 of the adapter"
```

**Example input / output:**

```

Input: Evaluate adapter-v2.3 for regression.

Output:

PYTHON
Copy
if is_regression("adapter-v2.3"):
    rollback_to_tag("adapter-v2.0")
```

**Data:**

```plaintext
/model-artifacts/
    ├── adapter-v1.0.tar.gz
    ├── adapter-v2.0.tar.gz
    └── adapter-v2.3.tar.gz
```

