---
type: "practice"
difficulty: "Intermediate"
---

# Why might you commit a small sample of your dataset to the repo while keeping the full dataset external?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Experiments with Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Experiments_with_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as an ML engineer, it's crucial to balance version control and manageability with data security and compliance. One effective practice is committing a small sample of your dataset to the repository while keeping the full dataset external. This approach has multiple benefits, including maintaining code integrity, ensuring reproducibility, and adhering to data privacy regulations.

This question asks why you would commit a small sample of your dataset to GitHub while keeping the full dataset on an external storage service or system. Let's break down the reasoning from first principles.

**Code:**

```bash
# Example command for committing a small sample to Git
git add sample_data.csv
git commit -m "Add small sample of training data"
```

