---
type: "practice"
difficulty: "Intermediate"
---

# What is a Git tag, and why might you tag the exact commit of the pipeline code that produced the numbers in a monthly reporting run?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Tags and Releases](<../../../../TOOLS/Git_and_GitHub/Git_Core/Tags_and_Releases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A Git tag is a reference to a specific commit in your version control history, typically used to mark significant milestones or release points. Tagging an exact commit of pipeline code ensures reproducibility and traceability, making it easier to pinpoint the state of your system at a particular point.

#### Triaging the Question
The question asks about Git tags specifically in the context of tagging commits that produced numbers from a monthly reporting run. The interviewer is probing your understanding of version control concepts, particularly how they apply to reproducibility and traceability in data pipelines. Assumptions include familiarity with Git and its use cases.

#### First-Principles Solution

Understanding Git Tags: A tag in Git is a reference to a specific commit hash, similar to branches but immutable once created. Unlike branches, which can move as new commits are added, tags remain fixed.
Importance of Reproducibility: In data pipelines, reproducibility means being able to rerun the exact same steps and get identical results. Tagging the pipeline code ensures that you can trace back exactly what version of the code was used for a particular reporting run.
Traceability: Tags help in maintaining a clear history of changes and their impact on outputs. This is crucial for debugging issues or understanding how certain data transformations were applied.

Step-by-Step Solution:

Identify the commit hash that produced the final numbers during the monthly reporting run.
Create a Git tag pointing to this specific commit using git tag -a <tag-name> <commit-hash> and push it to your remote repository if you haven't already done so.
Document the creation of the tag in your release or deployment notes, noting the date and any relevant details about the run.

#### Closing Summary
+ Common Mistakes/Follow-ups:

Mistakes: Forgetting to document tags properly can lead to confusion later when trying to trace back exactly what version was used for a particular output.

#### Follow-Ups
Discuss how continuous integration/continuous delivery (CI/CD) pipelines can automate the tagging process, ensuring consistency and reducing human errors. Also, consider best practices for naming conventions for tags.

**Code:**

```bash
# Example of creating and pushing a tag to remote repository
git tag -a v1.0.0 9b4d5f7 # Replace with actual commit hash
git push origin v1.0.0
```

