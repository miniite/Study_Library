---
type: "practice"
difficulty: "Deep"
---

# Your GITHUB_TOKEN in the ML repo has broad write access, and a compromised action could push a malicious model artifact. Walk through least-privilege token scoping per job and pinning actions to SHAs to shrink the supply-chain blast radius.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Token Permissions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Token_Permissions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the risk of a compromised GITHUB_TOKEN with broad write access in an ML repo, we need to implement least-privilege principles and minimize the potential damage from such a breach by controlling the actions and their execution context. Specifically, we will scope tokens per job, use action pinning via SHAs, and carefully manage secrets.

#### Triage

The core of the question is about ensuring that the GITHUB_TOKEN has only the necessary permissions for specific tasks to prevent unauthorized changes in the codebase or model artifacts. This involves:

Least-Privilege Token Scoping: Limiting token access to the exact set of operations required.
Action Pinning via SHAs: Ensuring actions are executed against a known, trusted version of the repository.

#### First-Principles Solution
Token Scoping:
Create dedicated tokens with minimal permissions for each job (e.g., deploy-model, train-model).
Use GitHub's token scopes to grant only necessary access rights.
Action Pinning via SHAs:
Define and pin actions to specific commit hashes (SHAs) in the workflow YAML files.
Ensure that actions are executed against these pinned commits, preventing them from running on subsequent changes.
Detailed Steps

Token Scoping

For each job, create a new GitHub token with minimal permissions required for that task.
Configure repositories to use these tokens exclusively for the respective jobs.

Action Pinning via SHAs

Identify the commit hash (SHA) of the last known good state in your repository.
In the workflow YAML files, pin actions to this SHA using env or runs-on directives if necessary.

#### Example
snippet:
YAML
jobs:
  train-model:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: <SHA-of-good-commit>
      - name: Train Model
        env:
          GITHUB_TOKEN: ${{ secrets.TRAIN_MODEL_TOKEN }}
        run: python train.py

Secret Management

Use GitHub Secrets to securely store sensitive information such as API keys or passwords.
Ensure that tokens and other credentials are stored in secure vaults and accessed only through these secrets.

Monitoring & Auditing

Set up monitoring for unusual activities or unauthorized changes.
Regularly audit token usage and action execution logs.

#### Closing Summary

By implementing least-privilege token scoping and pinning actions to specific SHAs, we can significantly reduce the risk of supply-chain attacks in an ML repository. This approach ensures that each job operates with minimal permissions and only executes against a known good state, minimizing the blast radius if a token is compromised.

#### Common Mistakes
& Follow-Ups
Mistake 1: Using a single, broad-access GITHUB_TOKEN for all operations.

#### Follow-Up
Always create separate tokens for each job with minimal required permissions.
Mistake 2: Not pinning actions to specific SHAs, leading to potential execution on unauthorized changes.

#### Follow-Up
Pin actions to known good SHAs in the workflow configuration.

**Code:**

```yaml
# Example Workflow YAML snippet

jobs:
  train-model:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: <SHA-of-good-commit>
      - name: Train Model
        env:
          GITHUB_TOKEN: ${{ secrets.TRAIN_MODEL_TOKEN }}
        run: python train.py
```

**Data:**

```yaml
# Example Workflow YAML snippet for pinning actions to SHAs

jobs:
  train-model:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        with:
          ref: <SHA-of-good-commit>
      - name: Train Model
        env:
          GITHUB_TOKEN: ${{ secrets.TRAIN_MODEL_TOKEN }}
        run: python train.py
```

**Flow:**

1. Yes
2. No
3. Checkout code
4. Pin to SHA?
5. train-model
6. Use latest commit
7. Train Model with restrictedtoken
8. Train Model with broaderaccess

