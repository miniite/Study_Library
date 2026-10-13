---
type: "practice"
difficulty: "Basic"
---

# Why is it risky to run pip install without a pinned requirements file committed to Git in a shared ML project?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Versioning Experiments with Git](<../../../../TOOLS/Git_and_GitHub/Git_Core/Versioning_Experiments_with_Git.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on an ML project, especially in a shared environment, running pip install without a pinned requirements file and committing it to Git can lead to several risks that might cause issues down the line. This approach is risky because:

Dependency Version Inconsistency: Different team members may have different versions of installed dependencies, which could lead to version conflicts.
Reproducibility Issues: Without a pinned requirements file, recreating the project environment on another machine or at a later time can be challenging due to differing dependency versions.

To understand why this is problematic, let's break it down from first principles:

#### Triage
What the Question Is Asking and Assumptions

The question is about understanding the risks of running pip install without ensuring that all dependencies are pinned in a requirements file. This typically involves assumptions such as:

The project uses Python packages.
There are multiple team members working on the same codebase.
The goal is to maintain consistency and reproducibility across different environments.

#### First-Principles Solution
Why Pinned Requirements Matter
Dependency Management: In any software project, dependencies can have breaking changes between versions. For example, a new version of a package might introduce bugs or incompatibilities with other packages.
Reproducibility: A pinned requirements file ensures that everyone working on the project has the exact same environment. This is crucial for reproducing results and ensuring the integrity of experiments.
Detailed Explanation

Dependency Version Conflict:

#### Example Scenario
Suppose you have a package numpy with version 1.20.3 installed. Later, your team decides to use a more recent version, say 1.23.0. If someone else on the team runs pip install, they might get an updated version, leading to unexpected behavior in the code.
Risk: Changes in dependencies can lead to subtle bugs that are difficult to track down and fix.

Reproducibility:

#### Example Scenario
Imagine you have a machine learning model trained with scikit-learn version 1.0.0. If someone else on the team runs pip install scikit-learn, they might get a different version, say 1.0.2. This can lead to slightly different results when running experiments.
Risk: Differences in dependency versions can cause your code to produce inconsistent results across different environments, making it difficult to compare outcomes and troubleshoot issues.

#### Closing Summary

Committing a pinned requirements.txt or environment.yml file to Git is crucial for ensuring that all team members have the same dependencies installed. This helps maintain consistency and reproducibility in the project environment.

#### Common Mistakes/Follow-Ups
Not Using Pinned Dependencies: Failing to commit a requirements file can lead to version mismatches.
Lack of Environment Management Tools: Not using tools like conda or pip freeze can make it harder to manage dependencies effectively.
Ignoring Dependency Updates: Regularly updating and pinning dependencies is important but often overlooked.

**Code:**

```python
# Example requirements file
numpy==1.20.3
scikit-learn==1.0.0
```

