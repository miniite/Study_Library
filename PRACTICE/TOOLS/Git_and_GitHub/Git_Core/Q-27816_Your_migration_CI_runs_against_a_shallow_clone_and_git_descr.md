---
type: "practice"
difficulty: "Intermediate"
---

# Your migration CI runs against a shallow clone and git describe fails, so the schema-version string baked into the release comes out wrong. Explain why depth-limited clones lose tag reachability and how you'd fix versioning without cloning full history.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Shallow Clones in CI](<../../../../TOOLS/Git_and_GitHub/Git_Core/Shallow_Clones_in_CI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand this issue, we need to first clarify what a shallow clone is and why it loses reachability to tags that are not fetched during the initial cloning process. Afterward, I'll explain how Git handles versioning in general, the implications of using a shallow clone for CI pipelines, and propose a solution.

A shallow clone in Git limits the history depth by only fetching a specified number of commits from the remote repository. This can significantly reduce the amount of data needed to clone a large project. However, this comes at the cost of not being able to fetch tags that are located beyond the shallow limit. When git describe is run against such a shallow clone, it fails because it cannot resolve the tag references.

Triaging the Problem

The core issue here is understanding why depth-limited clones lose reachability to certain tags and how we can version our database migrations without needing to clone full history. This involves both Git's underlying mechanics and strategic thinking about how to handle versioning in CI pipelines.

#### First-Principles Solution

To address this, let's break down the problem step-by-step:

Understand Tag Fetching:

When you create a shallow clone, only the latest commit is fetched along with its tags that are reachable from that commit.
Tags beyond this initial history depth are not automatically fetched and thus become unreachable.

Why git describe Fails:

The command git describe looks for specific tags (like "v1.0") in the repository’s history to determine the current branch's position relative to those tags.
If these tags are not reachable due to a shallow clone, git describe cannot find them and fails.

Versioning Strategy:

To fix this issue, we need to ensure that critical tags (like migration versions) can be fetched reliably during CI runs.
One approach is to modify the Git fetch strategy in your CI pipeline configuration to always fetch all relevant tags.
Detailed Solution

Here’s how you can address the versioning problem:

Adjust Fetch Strategy: Modify your .gitconfig or ~/.gitconfig file on the build server to include a custom fetch refspec that ensures all tags are fetched, even if using shallow cloning.

INI
[fetch]
    +refs/tags/*:refs/tags/*

Custom CI Script: Create a custom CI script that runs git fetch --tags before running your migration CI task. This ensures all necessary tags are available.

Use API for Tag Fetching: Alternatively, you can use the Git API to programmatically fetch and set the version string within your CI pipeline.

#### Example
CI Pipeline

Here’s a simplified example of how this might look in a CI configuration file (e.g., Jenkinsfile):

GROOVY
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git url: 'https://github.com/your-repo.git', depth: 1 // Shallow clone
                }
            }
        }

        stage('Fetch Tags') {
            steps {
                script {
                    sh 'git fetch --tags'
                }
            }
        }

        stage('Apply Migrations') {
            steps {
                script {
                    // Your migration logic here, e.g., calling a Python script or using a database tool
                }
            }
        }
    }
}

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the issue arises from shallow clones losing reachability to certain tags. By ensuring tags are always fetched during CI runs via git fetch --tags or through an API approach, we can reliably version our database migrations without needing a full clone.

#### Common mistakes
include not understanding how Git handles shallow clones and failing to adjust the fetch strategy properly. Always test your CI pipeline thoroughly with both deep and shallow clones to ensure consistent behavior.

When asked follow-up questions like “What if you have multiple branches?” or “How do you handle large repositories?”, consider strategies such as using Git LFS (Large File Storage) for very large repositories, or ensuring the relevant tags are present even in a shallow clone by adjusting your workflow and CI configuration accordingly.

**Code:**

```
GROOVY
Copy
pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    git url: 'https://github.com/your-repo.git', depth: 1 // Shallow clone
                }
            }
        }

        stage('Fetch Tags') {
            steps {
                script {
                    sh 'git fetch --tags'
                }
            }
        }

        stage('Apply Migrations') {
            steps {
                script {
                    // Your migration logic here, e.g., calling a Python script or using a database tool
                }
            }
        }
    }
}
```

