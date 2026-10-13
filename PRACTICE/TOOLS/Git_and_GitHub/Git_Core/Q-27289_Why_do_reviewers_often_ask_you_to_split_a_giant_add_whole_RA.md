---
type: "practice"
difficulty: "Intermediate"
---

# Why do reviewers often ask you to split a giant 'add whole RAG pipeline' commit into smaller commits for ingestion, retrieval, and generation?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Commit Messages](<../../../../TOOLS/Git_and_GitHub/Git_Core/Commit_Messages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with large code changes in an AI project involving a Retrieve-Augment-Generate (RAG) pipeline, it's common for reviewers to request splitting the commit into smaller ones. This practice enhances code readability, maintainability, and facilitates easier review processes. The key idea is that each small commit should address a single, well-defined change, making it simpler to understand the impact of individual modifications.

To fully grasp why this is done, let's break down the concepts involved:

Commit Size: Large commits can be overwhelming for reviewers, as they have to sift through extensive changes.
Isolation of Changes: Smaller commits help in isolating specific features or bug fixes, making it easier to trace and debug issues.
Revertability: If a small feature fails integration tests or introduces bugs, reverting individual commits is straightforward.

#### Triaging the Question

The question is fundamentally about understanding why reviewers prefer breaking down large code changes into smaller, more manageable chunks. The assumptions here are that:

The reviewer is looking for clarity and simplicity in the commit history.
The developer should follow best practices to ensure code quality and maintainability.
Large commits can be difficult to understand or revert if issues arise.

#### First-Principles Solution

Let's consider a typical RAG pipeline commit that includes multiple stages: ingestion, retrieval, and generation. A single large commit might look like:

GIT
commit 1234567890abcdef1234567890abcdef12345678
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 00:00:00 2023 +0000

    Add the entire RAG pipeline: ingestion, retrieval, and generation

Now, let's break this down into smaller commits:

GIT
commit abcdef1234567890abcdef1234567890abcdef
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 00:00:00 2023 +0000

    Add ingestion for RAG pipeline

commit 9876543210abcdef1234567890abcdef12345678
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 01:00:00 2023 +0000

    Add retrieval for RAG pipeline

commit fefebcd9876543210abcdef1234567890abcdef
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 02:00:00 2023 +0000

    Add generation for RAG pipeline

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, splitting large commits into smaller ones is beneficial because:

Readability: Smaller commits are easier to understand.
Isolation: Each commit can be reviewed independently.
Revertibility: Changes can be easily reverted if issues arise.

#### Common mistakes
include:

Failing to properly document and test each small change.
Overcomplicating the process by breaking down too fine-grained, making the commit history overly complex.
Ignoring code review feedback that suggests splitting commits appropriately.

#### Follow-up
questions might include:

How do you handle conflicts when multiple team members are working on different parts of a large pipeline?
What tools or practices do you use to manage and track smaller commits effectively?

**Comparison:**

| Commit Hash | Message |
|-------------|---------|
| 1234567890abcdef1234567890abcdef12345678 | Add the entire RAG pipeline: ingestion, retrieval, and generation |
| abcdef1234567890abcdef1234567890abcdef    | Add ingestion for RAG pipeline |
| 9876543210abcdef1234567890abcdef12345678 | Add retrieval for RAG pipeline |
| fefebcd9876543210abcdef1234567890abcdef  | Add generation for RAG pipeline |

**Code:**

```
GIT
Copy
commit 1234567890abcdef1234567890abcdef12345678
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 00:00:00 2023 +0000

    Add the entire RAG pipeline: ingestion, retrieval, and generation

commit abcdef1234567890abcdef1234567890abcdef
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 00:00:00 2023 +0000

    Add ingestion for RAG pipeline

commit 9876543210abcdef1234567890abcdef12345678
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 01:00:00 2023 +0000

    Add retrieval for RAG pipeline

commit fefebcd9876543210abcdef1234567890abcdef
Author: Developer Name <developer@example.com>
Date:   Mon Jan 1 02:00:00 2023 +0000

    Add generation for RAG pipeline
```

**Example input / output:**

```text
Input:
- Initial commit message: "Add the entire RAG pipeline: ingestion, retrieval, and generation"

Output:
- First split commit: "Add ingestion for RAG pipeline"
- Second split commit: "Add retrieval for RAG pipeline"
- Third split commit: "Add generation for RAG pipeline"
```

**Data:**

```json
{
  "commit_history": [
    {
      "hash": "1234567890abcdef1234567890abcdef12345678",
      "message": "Add the entire RAG pipeline: ingestion, retrieval, and generation"
    },
    {
      "hash": "abcdef1234567890abcdef1234567890abcdef",
      "message": "Add ingestion for RAG pipeline"
    },
    {
      "hash": "9876543210abcdef1234567890abcdef12345678",
      "message": "Add retrieval for RAG pipeline"
    },
    {
      "hash": "fefebcd9876543210abcdef1234567890abcdef",
      "message": "Add generation for RAG pipeline"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initial Commit
4. Is the commit large?
5. Split into smaller commits
6. Ingestion commit
7. Retrieval commit
8. Generation commit
9. Commit is fine as is

