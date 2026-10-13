---
type: "practice"
difficulty: "Intermediate"
---

# Your team wants conventional-commit messages enforced so changelogs auto-generate. How would you enforce the format with commitlint via a hook and in CI, and handle the friction juniors feel?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that conventional-commit messages are enforced in your team's repository, leveraging commitlint both as a Git pre-commit hook and within Continuous Integration (CI) is essential. This approach not only helps maintain consistent commit message formatting but also automates changelog generation. Here’s how you can achieve this while minimizing the friction for junior developers.

TL;DR
Install commitlint as a development dependency.
Configure commitlint to use conventional-commit rules.
Set up a Git pre-commit hook to run commitlint.
Integrate commitlint into your CI pipeline.
Educate and support junior developers with clear guidelines.

#### Triaging the Question

The question is primarily about enforcing a specific commit message format using commitlint. The core objective is to ensure consistent, structured commit messages that can later be used for automated changelog generation. The interviewer is probing on how you would implement this in both Git and CI contexts, as well as how you would manage the potential resistance or learning curve among junior team members.

#### First Principles Solution

Install commitlint: Begin by installing commitlint via npm:

SH
npm install commitlint commitlint-config-conventional -D

Configure Commitlint Rules: Use a conventional-commit rule set to enforce the desired format. Create a configuration file, typically named .commitlintrc.json, with rules such as type-enum, scope-allow, etc.

Set Up Git Pre-Commit Hook: To ensure that commits pass the commitlint check before being committed, create a pre-commit hook:

SH
# .git/hooks/pre-commit
npx commitlint --from="HEAD~1"

Integrate with CI Pipeline: Use your CI tool (e.g., GitHub Actions) to run commitlint as part of the build process.

Educate and Support Junior Developers: Provide clear, concise documentation and regular training sessions to help junior developers understand the benefits and best practices of using conventional-commit messages.

Detailed Steps

Install Dependencies:

SH
npm install commitlint commitlint-config-conventional -D

Create Commitlint Configuration: Create a .commitlintrc.json file with your desired rules:

JSON
{
  "extends": ["@commitlint/config-conventional"]
}

Set Up Git Pre-Commit Hook: In the root of your repository, create or edit the pre-commit file in the .git/hooks/ directory:

SH
#!/bin/sh

npx commitlint --from="HEAD~1" || exit 1

Configure CI Pipeline (example with GitHub Actions): Add a step to your GitHub Actions workflow that runs commitlint during the build process:

YAML
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Install dependencies
        run: npm ci
      - name: Run commitlint in CI
        run: npx commitlint --from="HEAD~1"

Educate Team Members: Create a wiki page or documentation that explains the benefits of conventional-commit messages and how to use them effectively. Schedule regular training sessions to ensure everyone is on board.

#### Closing Summary
and Common Mistakes

By setting up commitlint both as a pre-commit hook and in your CI pipeline, you can enforce consistent commit message formats while automating changelog generation. To handle the friction for junior developers, provide clear guidelines, offer support, and educate them about the importance of structured commits.

#### Common Mistakes

Forgetting to configure commitlint properly.
Not setting up a pre-commit hook.
Ignoring feedback from juniors or not providing sufficient training.
Failing to integrate commitlint in your CI pipeline effectively.

**Comparison:**

```
Step	Action
1	Install commitlint and dependencies.
2	Configure .commitlintrc.json.
3	Create pre-commit hook in .git/hooks/.
4	Integrate commitlint into CI pipeline.
5	Educate junior developers on best practices.
```

**Flow:**

1. Training
2. Repository
3. pre-commit Hook
4. commitlint --from='HEAD~1'
5. Commit Successful
6. CI Pipeline
7. Changelog Generation
8. Wiki Documentation
9. Regular Training Sessions
10. Junior Feedback Collection

