---
type: "practice"
difficulty: "Intermediate"
---

# You want a pre-commit hook that blocks committing any file over 50 MB or a notebook with non-empty output cells, but a teammate committed via a client without the hook. Why can't client hooks be a real gate, and how do you enforce it server-side or in CI?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Server-side Hooks and Push Protection](<../../../../TOOLS/Git_and_GitHub/Git_Core/Server-side_Hooks_and_Push_Protection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of committing large files and notebooks with non-empty output cells, we need to ensure that these checks are consistently enforced across all commit processes. Client hooks alone cannot reliably enforce such policies because they depend on the client-side execution environment, which may not be consistent or properly configured.

Triaging the Problem

The question revolves around why client-side pre-commit hooks might fail as a reliable enforcement mechanism and how to implement server-side or CI enforcement instead.

Client Hooks: These are scripts that run locally before a commit. They can provide a convenient way to run checks, but their effectiveness depends on the local environment and setup.
Server-Side Checks: These run on the Git server and can enforce policies consistently across all clients, regardless of the client's configuration.

#### First Principles Solution

To ensure that large files and notebooks with non-empty output cells are not committed, we need to implement the following steps:

Client-Side Pre-Commit Hook (Optional but Recommended):
This hook can run local checks and provide immediate feedback.
Server-Side Git Hooks:
Implement a pre-receive or update hook on the Git server that rejects commits containing large files or notebooks with non-empty output cells.
CI Pipeline:
Add steps in your CI pipeline to enforce these rules, ensuring they are checked before merging into main branches.

#### Implementation
Steps

Client-Side Pre-Commit Hook

BASH
# Example pre-commit hook script
# This is just a placeholder and needs to be adapted for the specific environment

#!/bin/bash
set -e

# Check for large files
git ls-files --others --exclude-standard | xargs -n 1 du -h | grep -E "^[0-9]+[MG]$" | awk '{print $1}' | while read size; do
    if [ "$(echo "$size" | cut -d'G' -f1)" -ge 50 ]; then
        echo "Error: Large file detected ($size). Please reduce the file size."
        exit 1
    fi
done

# Check for non-empty output cells in notebooks
git ls-files --others --exclude-standard | grep '\.ipynb$' | xargs -n 1 python3 notebook_check.py

Server-Side Pre-Receive or Update Hook

BASH
# Example pre-receive hook script
# This is just a placeholder and needs to be adapted for the specific environment

#!/bin/bash

while read oldrev newrev refname; do
    git --work-tree=/path/to/repo --git-dir=/path/to/git/dir diff-tree -r --name-only $oldrev $newrev | while read file; do
        if [ "$(du -b "$file" | cut -f1)" -ge 52428000 ]; then
            echo "Error: Large file detected in commit ($file). Please reduce the file size."
            exit 1
        fi
        # Check for non-empty output cells in notebooks
        if [[ $file =~ \.ipynb$ ]]; then
            python3 notebook_check.py "$file"
            if [ $? -ne 0 ]; then
                echo "Error: Notebook with non-empty output detected ($file). Please clean up the notebook."
                exit 1
            fi
        fi
    done
done

CI Pipeline

Integrate these checks into your CI pipeline, running them before merging to main branches.

#### Example
GitHub Actions YAML:
YAML
name: Enforce Commit Policies

on:
  push:
    branches: [ "main" ]

jobs:
  enforce-commit-policies:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run pre-commit hook locally
        run: ./scripts/pre_commit_hook.sh
        env:
          GITHUB_WORKSPACE: ${{ github.workspace }}

      - name: Check for large files and notebooks with non-empty output cells
        run: |
          git ls-files --others --exclude-standard | xargs -n 1 du -h | grep -E "^[0-9]+[MG]$" | awk '{print $1}' | while read size; do
              if [ "$(echo "$size" | cut -d'G' -f1)" -ge 50 ]; then
                  echo ::set-output name=large_file::true
                  exit 1
              fi
          done
          git ls-files --others --exclude-standard | grep '\.ipynb$' | xargs -n 1 python3 notebook_check.py

      - name: Fail if large files or notebooks with non-empty output cells are detected
        run: |
          if [ "${{ steps.check_large_files.outputs.large_file }}" == "true" ]; then
              exit 1
          fi

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing both client-side hooks and server-side Git hooks, along with integrating these checks into your CI pipeline, you ensure that large files and notebooks with non-empty output cells are consistently blocked. Common mistakes include relying solely on client hooks or not properly setting up the server-side hooks.

Client Hook Reliance: Client-side hooks can be disabled or improperly configured.
Server-Side Enforcement: Ensure the server-side hooks are correctly placed in .git/hooks and executable.
CI Pipeline Integration: Make sure CI steps are well-defined and run reliably, particularly if using GitHub Actions.

If you have any specific tools or frameworks that your team uses, these can be integrated into the above steps to provide a more tailored solution.

