---
type: "practice"
difficulty: "Intermediate"
---

# Why do open-source projects prefer a fork-and-PR model over giving everyone push access?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In open-source projects, the fork-and-pull-request (fork-and-PR) model is preferred to granting full push access to contributors because it enhances code quality, security, and community governance. This model allows for a structured workflow where developers can propose changes by creating pull requests (PRs), which are then reviewed and merged by maintainers or core contributors.

The fork-and-PR process fundamentally differs from giving everyone push access in that it introduces a layer of review and validation before any code is integrated into the main repository. This is crucial for maintaining project integrity, reducing security risks, and ensuring that the codebase remains high-quality. Here’s a step-by-step explanation:

Forking the Repository: A contributor forks the original repository to their own GitHub account. They can then make changes, add new features, or fix bugs without affecting the main project.

Creating Pull Requests (PRs): Once the desired changes are made, the contributor creates a pull request back to the original repository. This PR is visible to all maintainers and contributors, allowing them to review the code.

Review Process: Reviewers can discuss, suggest changes, or approve the PR based on its quality and fit with the project’s goals. This ensures that only high-quality code is merged into the main branch.

Merging Changes: After thorough review and approval, maintainers merge the approved PRs into the main repository.

Let's now delve into why this model works better than giving everyone push access:

Security Concerns: Allowing anyone to directly modify the main repository can introduce security risks. If a malicious actor gains unauthorized push access, they could potentially harm the project by making harmful or unwanted changes. The PR model mitigates these risks because it requires explicit review and approval.

Code Quality Assurance: Pull requests allow for a formal code review process, which helps maintain high standards of quality. Code reviewers can provide feedback that might improve the submitted code further, leading to better overall project outcomes.

Community Governance: By controlling who gets push access, maintainers can ensure that only trusted contributors have direct access to core repositories. This fosters a sense of community and responsibility among contributors.

Project Integrity: The review process helps prevent accidental or intentional damage to the codebase. It ensures that any changes are thoroughly tested and approved before being committed to production.

Transparency and Collaboration: Pull requests provide visibility into all proposed changes, fostering transparency and enabling better collaboration within the community. This transparency can also help in maintaining a positive and healthy contributor ecosystem.

Easier Reverting Changes: If a PR is rejected or issues arise post-merge, it's much easier to revert the changes made via a pull request compared to directly modifying the main repository with push access.

#### Closing Summary
The fork-and-PR model provides a balanced approach by allowing open collaboration while maintaining security and code quality. It ensures that contributions are reviewed, validated, and merged systematically, contributing to a more robust and reliable project.

#### Common Mistakes/Follow-Ups

Ignoring the importance of the review process.
Assuming all contributors are equally trustworthy or have the same level of expertise.
Not providing clear guidelines on how pull requests should be structured and reviewed.

