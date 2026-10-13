---
type: "practice"
difficulty: "Deep"
---

# You pin a Hugging Face model by branch name (main) in your inference code and a silent model update broke output format overnight. Why is pinning by branch dangerous and how do you pin by immutable revision?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Pin-by-branch in Git and GitHub can be dangerous because branches are mutable; they change over time as new commits are added, potentially breaking your code or causing inconsistencies in outputs. To avoid such issues, you should pin models by immutable revisions (commit hashes). This ensures that the exact version of the model used during inference remains constant.

The question is probing whether you understand the concept of mutable vs. immutable references and the importance of immutability for reproducibility. The assumptions here include knowledge of Git/GitHub workflows, understanding of version control systems, and familiarity with Hugging Face models and their updates.

Step-by-Step Solution

Understanding Mutable Branches:

A branch in Git is a pointer to the latest commit in your repository.
When you fetch or pull changes from a remote (e.g., main), it can change this pointer, potentially altering the state of the model you are using.

Implications for Model Output:

If you pin by branch name, any update to that branch will automatically affect your inference code.
This means if someone pushes changes to the main branch (e.g., fixing a bug or adding new features), your application might start producing different outputs than expected.

Pin-by-Revision:

Instead of pinning by branch, you should use the commit hash of the exact version you need.
This ensures that even if changes are pushed to main, the model used for inference remains unchanged until you explicitly update your code.

Practical Implementation:

For Hugging Face models, you can download a specific revision using the model identifier and the commit hash. Here’s an example in Python:
PYTHON
from transformers import pipeline

# Example of pinning by revision
model_id = "bert-base-uncased@commit_hash"
nlp = pipeline("text-classification", model=model_id)
Replace commit_hash with the actual hash of the desired commit.

Handling Future Updates:

If you decide to update the model, ensure that any new revisions are backward compatible or thoroughly tested before updating the application.
Use Git tags for stable versions if available and follow a release management strategy.

#### Closing Summary

Pin-by-branch can lead to unexpected behavior due to mutable nature of branches. To avoid issues like silent model updates breaking your output format, pin models by immutable revisions (commit hashes). This approach ensures that the exact version used remains constant, maintaining reproducibility and stability in applications.

#### Common Mistakes/Follow-Ups
Mistake: Using main branch for pinning without considering potential future changes.

#### Follow-Up
Discuss strategies like using tags or branches with specific labels to manage different versions of models more effectively.

**Code:**

```python
from transformers import pipeline

# Example code snippet for pinning by revision
model_id = "bert-base-uncased@commit_hash"
nlp = pipeline("text-classification", model=model_id)
```

