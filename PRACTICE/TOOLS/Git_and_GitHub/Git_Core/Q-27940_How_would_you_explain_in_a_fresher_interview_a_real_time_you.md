---
type: "practice"
difficulty: "Basic"
---

# How would you explain, in a fresher interview, a real time you used branches and pull requests on a college data-engineering or analytics project?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pull Requests](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pull_Requests.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a fresh perspective of explaining using Git and GitHub during a college data-engineering project, let's dive into an example scenario where we utilized branches and pull requests to manage our codebase effectively.

TL;DR: In this project, we used branches for parallel development and pull requests (PRs) for code review and merging, ensuring the quality of changes before they reach the main branch. This approach helped us maintain a clean commit history and avoid conflicts in our data processing pipeline.

#### Triage

The question is asking to explain how branches and pull requests were used in a real project during college, focusing on why these tools are essential for collaboration and code management in a team setting. We need to assume the context of a college project where multiple students worked on different parts of the same data-processing pipeline.

#### First-Principles Solution

In our college project, we were tasked with building a real-time analytics dashboard for tracking student performance. The team consisted of four members, each responsible for different components: data ingestion, preprocessing, transformation, and visualization. We used Git and GitHub to manage our codebase effectively.

Branch Creation: Each member created their own branch based on the main branch (often called main or master). For example, a student working on the preprocessing step would create a branch named preprocessing-optimization.

Development in Branches: Developers worked on their respective branches independently, making changes and commits as needed. This allowed parallel development without blocking each other.

Code Review via Pull Requests: Once a developer finished their work, they created a pull request (PR) to merge their branch back into the main branch. The PRs were reviewed by at least one peer to ensure that the code was clean, followed best practices, and did not introduce bugs or conflicts.

Merge Strategy: After approval, the changes from the PR would be merged into the main branch. This process helped maintain a single source of truth for our project, ensuring that all team members were working with the latest version of the code.

#### Closing Summary

By using branches and pull requests, we effectively managed parallel development, ensured quality through peer reviews, and maintained a clean commit history. This approach not only helped in avoiding merge conflicts but also promoted better collaboration and accountability among team members.

#### Common Mistakes/Follow-Ups
Forgetting to Merge Changes: One common mistake is forgetting to merge changes from the main branch into individual branches. This can lead to outdated code and potential integration issues.
Insufficient Code Review: Another pitfall is not thoroughly reviewing PRs, which can result in introducing bugs or security vulnerabilities.
Overlooking Conflict Resolution: Conflicts are inevitable when multiple people work on the same file. Proper conflict resolution techniques should be discussed and followed.

In conclusion, using Git branches and pull requests was crucial for our college project's success. It streamlined collaboration, ensured code quality, and maintained a clean repository history.

**Comparison:**

| Branch | Developer | Status |
|--------|-----------|--------|
| Data Ingestion Improvements | Developer B | Merged |
| Preprocessing Optimizations | Developer C | Approved, needs changes |
| Transformation Enhancements | Developer D | Merged |

**Code:**

```
MERMAID
Copy
graph LR
    A[Main Branch] --> B1[Branch: Data Ingestion]
    A --> C1[Branch: Preprocessing]
    A --> D1[Branch: Transformation]
    A --> E1[Branch: Visualization]

    B1 --> B2[PR: Improvements in Data Ingestion]
    C1 --> C2[PR: Optimizations in Preprocessing]
    D1 --> D2[PR: Enhancements in Transformation]
    E1 --> E2[PR: Updates in Visualization]
```

**Example input / output:**

```text
sequenceDiagram
    participant DeveloperA as A
    participant DeveloperB as B
    participant DeveloperC as C
    participant DeveloperD as D

    A->>B: Create branch preprocessing-optimization
    A->>C: Create branch data-ingestion-improvements
    A->>D: Create branch transformation-enhancements

    B->>A: Push changes to branch preprocessing-optimization
    C->>A: Push changes to branch data-ingestion-improvements
    D->>A: Push changes to branch transformation-enhancements

    A->>B: Open PR for improvements in Data Ingestion
    A->>C: Open PR for Optimizations in Preprocessing
    A->>D: Open PR for Enhancements in Transformation

    B->>A: Review and approve PR from C
    C->>A: Request changes in PR
    D->>A: Merge PR after resolving conflicts
```

**Data:**

```json
{
  "branch_structure": {
    "main": [
      "data-ingestion",
      "preprocessing-optimization",
      "transformation-enhancements",
      "visualization-updates"
    ],
    "pull_requests": [
      { "title": "Improvements in Data Ingestion", "status": "merged" },
      { "title": "Optimizations in Preprocessing", "status": "approved" },
      { "title": "Enhancements in Transformation", "status": "pending" },
      { "title": "Updates in Visualization", "status": "rejected" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is a new feature or bug fix?
7. Create a new branch
8. Close the issue
9. Implement the changes
10. Test the changes
11. Open PR for code review
12. Review and merge PR?
13. Merge branch into main

