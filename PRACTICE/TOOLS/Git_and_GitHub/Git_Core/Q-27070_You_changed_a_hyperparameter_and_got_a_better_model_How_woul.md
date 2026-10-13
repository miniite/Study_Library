---
type: "practice"
difficulty: "Intermediate"
---

# You changed a hyperparameter and got a better model. How would you use Git commits to record which code state produced that model?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with machine learning models, it’s crucial to track changes in your codebase effectively, especially when hyperparameters are adjusted and the model performance improves. Using Git commits allows you to maintain a clear history of changes, making it easier to understand which specific modifications led to an improvement.

The core idea here is to create meaningful commit messages that reflect the changes made and their impact on the model's performance. This involves identifying the exact lines or files modified, along with any relevant hyperparameters adjusted. Let’s break down how you would achieve this step-by-step from first principles.

#### Triage

The question asks about using Git commits to record which code state produced a better model after changing a hyperparameter. The interviewer is probing your understanding of version control and how to effectively document changes in a codebase, particularly in the context of machine learning projects. Key assumptions include familiarity with basic Git commands and the importance of clear commit messages.

#### Solution

Identify the Changed Code: First, identify which lines or files were modified when changing the hyperparameter. For example, if you changed the batch size from 32 to 64 in a training script, note this explicitly.

Run and Evaluate the Model: Ensure that running the model with these changes results in improved performance before committing any changes.

Create a Commit Message:

Descriptive and Clear: Write a clear commit message explaining what was changed and how it impacts the model. For example: "Updated batch size to 64, resulting in slightly better accuracy."
Include Relevant Information: If applicable, include links or references to any supporting files, such as performance metrics or logs.

Add and Commit Changes:

SH
git add <modified_file>
git commit -m "Updated batch size to 64, resulting in slightly better accuracy."

Push to GitHub: Finally, push the committed changes to your remote repository on GitHub.

#### Closing Summary
+ Common Mistakes/Follow-ups

To summarize, using Git commits effectively involves creating clear and descriptive messages that document changes in code states leading to improved model performance. This not only helps you but also future collaborators understand the evolution of the project.

#### Common mistakes
include:

Failing to add relevant files before committing.
Writing vague or uninformative commit messages.
Overlooking the need for thorough testing after making significant changes.

#### Follow-ups
might involve discussing strategies for branching and merging, how to handle multiple experiments simultaneously, and best practices for integrating these changes into a larger workflow.

