---
type: "practice"
difficulty: "Intermediate"
---

# You want PR authors to run prompt-lint and an eval smoke-test before pushing, but Git hooks aren't shared by default. How do you distribute them via core.hooksPath or Husky, and why still re-run the same checks in CI?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pre-commit Hooks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pre-commit_Hooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that PR authors run prompt-lint and an eval smoke-test before pushing code to a Git repository, we can leverage Git hooks through core.hooksPath or use Husky as a wrapper around these hooks. This approach ensures that the necessary checks are performed locally by developers, enhancing the quality of their contributions without relying solely on CI pipelines.

#### Triage
Understanding the Problem

The key aspects of this question are:

Locally Enforcing Checks: We need to make sure that prompt-lint and eval smoke-test are run before code is pushed.
Shared Hooks: By using core.hooksPath or Husky, we can ensure these hooks are shared across developers' machines.
Re-running in CI: Even though local checks exist, they should still be re-run in CI to catch any issues that might not have been caught locally.

#### First Principles Solution
Step 1: Set Up Local Git Hooks Using core.hooksPath

core.hooksPath allows us to specify a directory where our custom hooks are located. This is ideal for sharing common hooks across multiple repositories or teams.

Create Custom Hooks Directory: In your project root, create a directory named .git/hooks.
Write the Hooks:
pre-push: This hook runs before code is pushed and can check that prompt-lint and eval smoke-test have been run.
SH
# .git/hooks/pre-push
#!/bin/sh

# Run local checks
npm run lint:check && npm run test:smoke || exit 1

# Allow push if all checks pass
echo "Local pre-push hooks passed."
Ensure Hooks are Executable:
SH
chmod +x .git/hooks/pre-push
Step 2: Use Husky for Simplified Hook Management

Husky is a Git hook wrapper that simplifies the process of creating and managing pre-commit, commit-msg, pre-push, etc. hooks.

Install Husky:

SH
npm install husky --save-dev

Configure Husky:

Run npx husky install to install Husky.
Create a .huskyrc file or modify the existing one in your root directory.

Define Custom Hooks:

In the .husky/pre-push hook, ensure prompt-lint and eval smoke-test are run before pushing:
SH
# .husky/pre-push
#!/bin/sh

# Run local checks
npm run lint:check && npm run test:smoke || exit 1

# Allow push if all checks pass
echo "Husky pre-push hooks passed."
Step 3: Re-run Checks in CI

While local checks are important, they should still be re-run in CI to ensure thorough testing and catch any issues that might have been overlooked locally.

CI Pipeline Setup: Ensure your CI pipeline includes steps for running prompt-lint and eval smoke-test.

#### Example
CI Configuration (using GitHub Actions):
YAML
name: Check Commit

on:
  push:
    branches:
      - main

jobs:
  check-commit:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - run: npm install
      - run: npm run lint:check && npm run test:smoke

#### Closing Summary
+ Common Mistakes/Follow-Ups

By setting up custom Git hooks and leveraging Husky, we ensure that prompt-lint and eval smoke-test are performed locally before code is pushed. This not only enhances the quality of code contributions but also reduces CI load by catching issues earlier.

#### Common Mistakes

Not Making Hooks Executable: Ensure all hooks have execute permissions.
Over-relying on Local Checks: While local checks are crucial, they should still be re-run in CI to catch edge cases and ensure comprehensive testing.

#### Follow-Ups

Explore additional pre-commit or pre-push checks that might complement prompt-lint and eval smoke-test.
Investigate performance implications of running these checks locally versus relying on CI.

**Code:**

```
.git/hooks/pre-push

#!/bin/sh

npm run lint:check && npm run test:smoke || exit 1

CODE
Copy

```sh
# Husky pre-push hook
#!/bin/sh

npm run lint:check && npm run test:smoke || exit 1
```

