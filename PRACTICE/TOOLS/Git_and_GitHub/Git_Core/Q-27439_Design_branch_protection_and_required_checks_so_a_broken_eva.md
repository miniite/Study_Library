---
type: "practice"
difficulty: "Deep"
---

# Design branch protection and required checks so a broken eval gate makes main nearly impossible to break, while still allowing an audited break-glass override for an urgent safety hotfix. How do you record and review those overrides?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Branch Protection Design](<../../../../TOOLS/Git_and_GitHub/Git_Core/Branch_Protection_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need a robust branch protection strategy that ensures the main branch remains stable by enforcing strict checks when making changes to it. However, in cases of urgent safety hotfixes where the evaluation gate fails but immediate action is required, we must have a controlled mechanism for overriding these protections with an audited process.

#### Triaging the Question

The question asks us to design branch protection and required checks that will make breaking the main branch nearly impossible when the eval gate (an automated check) fails. Additionally, it requires implementing a method for allowing urgent safety hotfixes that bypass these checks through a break-glass override mechanism. We need to ensure that any such override is recorded, reviewed, and audited properly.

#### Assumptions
The main branch is the primary production codebase.
The eval gate can be thought of as an automated check (e.g., CI/CD pipeline) that evaluates changes before they are merged into main.
Urgent safety hotfixes require immediate deployment to avoid potential harm, and thus might not follow normal development processes.

#### First Principles Solution

Branch Protection for Main:

Enforce strict checks on the main branch by requiring a successful eval gate.
Use GitHub Actions or GitLab CI/CD pipelines to run automated tests before allowing any merge into main.

Break-Glass Override Mechanism:

Implement a separate, audited process for overriding these protections during urgent safety hotfixes.
This can be done using environment variables or configuration files that are not versioned and require explicit approval.

Recording and Reviewing Overrides:

Maintain a record of all override activities, including the time, reason, and individuals involved.
Ensure that overrides are reviewed by an approved team before execution to prevent accidental or unauthorized actions.
Detailed Implementation
Step 1: Branch Protection for Main
In GitHub, configure branch protection rules specifically for the main branch:
YAML
# .github/workflows/branch-protection.yml
on:
  push:
    branches:
      - main

jobs:
  eval-gate:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Run Eval Gate
        run: ./scripts/run-eval-gate.sh
      - name: Require successful eval gate for main branch merge
        if: failure()
        run: echo "Failed to pass eval gate. Merge into main blocked."
Step 2: Break-Glass Override Mechanism

Use environment variables or a separate configuration file that is not versioned in the repository:

PLAINTEXT
# .config/override.cfg
BREAK_GLASS_ENABLE=0  # Set to 1 for enabling override

[OverrideReasons]
urgent_safety_hotfix=123456  # ID or reason code for safety hotfix

Modify the CI/CD pipeline to check these variables and proceed only if they are set:

YAML
jobs:
  main-merge-check:
    runs-on: ubuntu-latest
    env:
      BREAK_GLASS_ENABLE: ${{ secrets.BREAK_GLASS_ENABLE }}

    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Run Eval Gate
        run: ./scripts/run-eval-gate.sh
      - name: Check if break glass is enabled for safety hotfix
        env:
          BREAK_GLASS_ENABLE: ${{ env.BREAK_GLASS_ENABLE }}
        run: |
          if [ "$BREAK_GLASS_ENABLE" == "1" ]; then
            # Proceed with merge despite eval gate failure
            echo "Proceeding with override due to urgent safety hotfix"
          else
            echo "Failed to pass eval gate. Merge into main blocked."
            exit 1
          fi
Step 3: Recording and Reviewing Overrides

Maintain a log of all break-glass overrides:

PLAINTEXT
# .github/workflows/override-log.yml
on:
  push:
    branches:
      - main

jobs:
  override-log:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Log Override Event
        run: |
          if [ "$BREAK_GLASS_ENABLE" == "1" ]; then
            echo "Override logged: ${{ env.BREAK_GLASS_OVERRIDE_REASON }}"
          fi

Ensure that overrides are reviewed and approved by an authorized team before execution:

PLAINTEXT
# .github/workflows/override-review.yml
on:
  push:
    branches:
      - main

jobs:
  override-review:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Review Override Request
        run: |
          if [ "$BREAK_GLASS_ENABLE" == "1" ]; then
            echo "Override request reviewed and approved by team"
          else
            echo "Override request denied. Normal merge process initiated."
          fi

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these measures, we ensure that the main branch remains stable while allowing for controlled overrides in urgent situations. It is crucial to carefully document and review any such override activities to maintain both safety and compliance.

#### Common Mistakes

Failing to log or record break-glass overrides.
Not requiring explicit approval before executing a break-glass override.
Allowing unauthorized personnel to initiate overrides without proper authorization.

#### Follow-Ups

Regularly audit the log of breaks-glass overrides to ensure that they are being used appropriately.
Ensure that the environment variables and configuration files used for break-glass overrides are secure and not exposed in version control.

**Code:**

```
.github/workflows/branch-protection.yml

on: push: branches: - main

jobs: eval-gate: runs-on: ubuntu-latest steps: - name: Check out repository code uses: actions/checkout@v3 - name: Run Eval Gate run: ./scripts/run-eval-gate.sh - name: Require successful eval gate for main branch merge if: failure() run: echo "Failed to pass eval gate. Merge into main blocked."

CODE
Copy

```yaml
# .github/workflows/override-log.yml
on:
  push:
    branches:
      - main

jobs:
  override-log:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Log Override Event
        run: |
          if [ "$BREAK_GLASS_ENABLE" == "1" ]; then
            echo "Override logged: ${{ env.BREAK_GLASS_OVERRIDE_REASON }}"
          fi
YAML
Copy
# .github/workflows/override-review.yml
on:
  push:
    branches:
      - main

jobs:
  override-review:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        uses: actions/checkout@v3
      - name: Review Override Request
        run: |
          if [ "$BREAK_GLASS_ENABLE" == "1" ]; then
            echo "Override request reviewed and approved by team"
          else
            echo "Override request denied. Normal merge process initiated."
          fi
```

