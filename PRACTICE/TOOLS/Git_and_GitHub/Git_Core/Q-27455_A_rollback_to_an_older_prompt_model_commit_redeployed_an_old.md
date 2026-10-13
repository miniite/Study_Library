---
type: "practice"
difficulty: "Expert"
---

# A rollback to an older prompt+model commit redeployed an old agent, but the vector index had already migrated to a new embedding space, breaking retrieval. How do you couple Git-driven rollbacks with index state so a code revert doesn't fight the index?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When rolling back to an older commit in your AI project using Git and GitHub, it's crucial to ensure that the vector index remains consistent with the application code. If you rollback the codebase but the vector index has already been updated to a newer embedding space, this inconsistency can break retrieval functionality. The key is to couple Git-driven rollbacks with an index state management strategy that ensures both are in sync.

To achieve this, we need to manage the lifecycle of the vector index and ensure it’s properly versioned and reverted when necessary. Let's walk through how we can accomplish this from first principles:

Understand What We’re Probing: The question is about maintaining consistency between code and the vector index during rollbacks. This involves ensuring that when you revert to an older commit, the vector index reverts to its state at the time of that commit.

#### Assumptions
You have a well-structured Git repository with clear commit history for both the application code and any relevant parts of the vector index.
The vector index is stored in a way that can be versioned and reverted, such as being part of the same Git repository or managed separately but linked.
Step-by-Step Solution
Version Control for Vector Index:
Ensure that the vector index updates are tracked alongside code changes. This could mean storing vector index snapshots (e.g., in a separate branch) that get updated whenever there’s a significant change.
Commit History Correlation:
Keep a detailed commit history where each vector index update is correlated with the corresponding application code commit. This can be done using Git tags or by explicitly noting changes in commit messages.
Rollback Strategy:
When rolling back to an older commit, automatically revert the vector index to its state at that point in time. This could involve a script or tool that parses commit history and updates the vector index accordingly.

#### Implementation
Details
1. Git Repository Structure
PLAINTEXT
ai-app/
├── app-code/
│   ├── v1_code/  # Older codebase
│   └── v2_code/  # Current codebase
├── vector-index-snapshots/
│   ├── v1_snapshot/index.pkl  # Vector index at commit A
│   └── v2_snapshot/index.pkl  # Vector index at latest commit B
└── .gitignore
2. Commit History and Tagging
Use tags or annotations in the commit messages to indicate when a vector index snapshot is created.
PLAINTEXT
# Example commit message for creating a new vector index snapshot
"feat: Update vector index with embeddings from v1_code - [index_snapshot:v1_snapshot]"
3. Rolling Back Code and Vector Index
Create a script that parses the Git history to identify relevant commits and their corresponding vector index snapshots.
BASH
#!/bin/bash

# Variables
GIT_REPO="path/to/ai-app"
INDEX_DIR="$GIT_REPO/vector-index-snapshots"

# Function to get commit hash from tag
get_commit_from_tag() {
    git describe --tags "$1" | cut -d '-' -f 2
}

# Rollback to a specific commit and its corresponding vector index snapshot
rollback_to_commit() {
    local commit=$1
    local index_tag=$(git describe --tags $commit | cut -d '-' -f 3)
    local index_snapshot="$INDEX_DIR/${index_tag}_snapshot/index.pkl"

    echo "Rolling back to commit: $commit, Index snapshot: $index_snapshot"

    # Revert application code
    git checkout $commit

    # Update vector index
    python update_vector_index.py --path-to-index "$index_snapshot"
}

# Example usage
rollback_to_commit "v1_code"

#### Closing Summary
and Common Mistakes

To summarize, the key is to maintain a versioned vector index that aligns with your application code. By doing so, rollbacks can be seamlessly managed without breaking the retrieval functionality.

#### Common mistakes
include:

Failing to keep detailed commit history and tags.
Not having a mechanism to automatically or manually update the vector index when necessary.
Ignoring the importance of keeping both code and data in sync during rollbacks.

**Comparison:**

| Commit | Index Snapshot |
|--------|----------------|
| v1_code | v1_snapshot    |
| v2_code | v2_snapshot    |

**Code:**

```bash
#!/bin/bash

# Function to get commit hash from tag
get_commit_from_tag() {
    git describe --tags "$1" | cut -d '-' -f 2
}

rollback_to_commit() {
    local commit=$1
    local index_tag=$(git describe --tags $commit | cut -d '-' -f 3)
    local index_snapshot="$INDEX_DIR/${index_tag}_snapshot/index.pkl"

    echo "Rolling back to commit: $commit, Index snapshot: $index_snapshot"

    git checkout $commit
    python update_vector_index.py --path-to-index "$index_snapshot"
}

rollback_to_commit "v1_code"
```

**Example input / output:**

```text
$ git checkout "v1_code"
$ python update_vector_index.py --path-to-index "vector-index-snapshots/v1_snapshot/index.pkl"
Rolling back to commit: v1_code, Index snapshot: vector-index-snapshots/v1_snapshot/index.pkl
```

**Data:**

```plaintext
ai-app/
├── app-code/
│   ├── v1_code/
│   └── v2_code/
├── vector-index-snapshots/
│   ├── v1_snapshot/index.pkl  # Vector index at commit A
│   └── v2_snapshot/index.pkl  # Vector index at latest commit B
└── .gitignore
```

**Flow:**

1. Yes
2. No
3. Rollback to v1_code
4. Is index snapshot available?
5. Revert app-code to v1_code
6. Update vector index withv1_snapshot
7. Manually create newv1_snapshot and then revert

