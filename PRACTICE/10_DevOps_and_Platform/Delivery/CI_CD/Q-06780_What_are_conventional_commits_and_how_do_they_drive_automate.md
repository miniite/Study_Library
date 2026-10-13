---
type: "practice"
difficulty: "Intermediate"
---

# What are conventional commits and how do they drive automated version bumps?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Event Driven Deploys](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Schema_Evolution_in_Event_Driven_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Conventional commits are a standardized approach to commit messages that allow CI/CD pipelines to automatically increment version numbers based on changes in source code. This automation helps maintain version control, traceability, and consistency across development processes. In this explanation, I'll cover the core idea of conventional commits, their implementation details, how they fit into CI/CD workflows, and some common pitfalls.

#### Triage
What's Being Probed

The question is probing your understanding of commit message standards in version control systems, specifically how these standards can be leveraged to automate version number bumps. The interviewer might also evaluate your ability to explain the benefits and trade-offs of adopting such practices.

#### Assumptions
You understand basic Git workflow.
You are familiar with CI/CD pipelines.
You have experience working with version control systems like SemVer (Semantic Versioning).

#### First Principles Solution
What Are Conventional Commits?

Conventional commits follow a specific pattern in commit messages to convey the type of change and other relevant details. This standardization enables tools to automatically parse these messages, infer the nature of changes, and update version numbers accordingly.

Commit Message Format

A typical conventional commit message follows this format:

CODE
<type>(<scope>): <subject>
<body>
<footer>
Type: Describes the type of change (e.g., feat, fix, docs).
Scope (optional): The component affected by the change.
Subject: A short summary of the change.
Body (optional): Detailed description of the changes.
Footer (optional): References to issues or other notes.

#### Example
Commit Message
CODE
feat(api): add user authentication

Add basic HTTP headers for JWT-based authentication in API requests.
Why Use Conventional Commits?
Automated Version Bumping: Tools like conventional-changelog can parse commit messages and automatically increment the version number based on the types of changes (e.g., feat, fix).
Traceability: Clear commit messages make it easier to understand what has changed.

#### Cons
istency: Standardized commit messages ensure everyone follows a consistent pattern.

#### Implementation
in CI/CD Pipelines

In your CI/CD pipeline, you can use tools like conventional-changelog and semantic-release to automatically detect changes based on the commit message patterns and bump version numbers accordingly.

Tool Installation: Install the required tools in your build environment.
Commit Hook Integration: Configure a Git hook or modify your CI/CD configuration to include these tools as part of the build process.
Version Bumping Logic: These tools will automatically read commit messages, parse them, and determine if version bumps are necessary.

#### Example
Workflow
Developer makes changes to code and commits with a conventional message.
conventional-changelog parses the commit messages during the CI/CD run.
If it detects a new feature or fix, it increments the version number accordingly.
The updated package is then published to the appropriate repository.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Conventional commits provide a structured way to manage and automate versioning in software development. While they offer significant benefits, they require consistent adherence by all team members. Misunderstandings or inconsistent use can lead to incorrect version bumps and other issues. During the interview, you might be asked about specific tools used, the impact on branching strategies, or how to handle edge cases like hotfixes.

**Comparison:**

```
Commit Message Type	Description
feat	A new feature
fix	A bug fix
docs	Documentation changes
refactor	Code refactoring that doesn't affect functionality
chore	Changes to the build process or auxiliary tasks
```

**Code:**

```bash
# Example of a conventional commit hook in Git
#!/bin/sh

# Check for conventional commits using `conventional-changelog` script
git log --oneline $1..HEAD | grep -E '^[feat|fix|docs]' > /dev/null

if [ $? -eq 0 ]; then
    echo "Conventional commits detected. Version will be bumped."
else
    echo "No conventional commits found. No version bump needed."
fi
```

**Example input / output:**

```text
Input:
- Commit message: "feat(api): add user authentication"
Output:
- Parsed data:
  - Type: feat
  - Scope: api
  - Subject: add user authentication
  - Body: (empty)
  - Footer: (empty)
```

**Data:**

```json
{
  "commit_message": "feat(api): add user authentication",
  "parsed_data": {
    "type": "feat",
    "scope": "api",
    "subject": "add user authentication",
    "body": "",
    "footer": ""
  }
}
```

**Flow:**

1. Yes
2. No
3. Developer Commits
4. Is Conventional Commit?
5. Conventional-changelogparses commit
6. Automatically incrementversion number
7. Version not bumped

