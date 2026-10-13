---
type: "practice"
difficulty: "Basic"
---

# What is a draft pull request on GitHub and when would you open one?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A draft pull request (PR) on GitHub is a feature that allows developers to create a PR without merging changes immediately, essentially serving as a staging area where code can be reviewed before it's ready for integration into the main branch. This is useful when you're still working on features or bug fixes but want early feedback from your team members.

#### Triage

GitHub introduces draft pull requests as part of its suite of collaboration tools to enhance the review process. When would you open a draft PR? You might do this when:

Code is in Progress: Your code isn't yet complete, and you need input before it's ready for merging.
Early Feedback Needed: There are design or architectural decisions that require discussion and approval before proceeding with the full implementation.

#### First-Principles Solution

To open a draft PR on GitHub:

Choose the Source Branch: Select the branch from which you want to initiate the PR (often your feature branch).
Create the Draft PR:
Go to the repository where your code lives.
Click on the Pull requests tab.
Click the New pull request button.
Choose the source and target branches.
Optionally, write a title and description for the PR.
Toggle the Draft Pull Request option to enable it.

Once you've created a draft PR, any changes made in the source branch will be reflected in the PR. You can push your code directly into this branch or continue working in other branches while keeping the PR updated with the latest changes. The draft status allows reviewers to provide feedback without the risk of merging incomplete features.

#### Closing Summary
+ Common Mistakes/Follow-Ups

A common mistake is opening a draft PR prematurely, which might lead to unnecessary discussions and delays. Always ensure that you have enough context or that your code is in a state where it can benefit from early feedback before marking it as a draft.

#### Follow-up
questions the interviewer might ask include:

How do you manage conflicts during a draft PR?
What are some best practices for using draft PRs effectively?
How does GitHub handle notifications and communication around draft PRs?

**Code:**

```
MERMAID
Copy
graph TD;
    A[Choose Source Branch] --> B[Create Draft PR];
    B --> C[Toggle 'Draft Pull Request' Option];
    C --> D[Update with Latest Changes];
```

**Flow:**

1. Choose Source Branch
2. Create Draft PR
3. Toggle 'Draft Pull Request'Option
4. Update with Latest Changes

