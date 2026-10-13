---
type: "practice"
difficulty: "Intermediate"
---

# Why are local Git hooks not shared by default, and how do tools like Husky get around that?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Pre-commit Hooks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Pre-commit_Hooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Local Git hooks aren't shared by default because they are designed to be specific to individual developers or environments. However, tools like Husky provide a way to make these local hooks executable globally within a project.

#### Triage
and Assumptions:
The question is probing the understanding of Git hooks and their limitations.
Local hooks run only on the machine where they were created, whereas global and system-wide hooks can be shared across multiple machines. The core issue is understanding why this separation exists and how Husky leverages it.

#### First-principles Solution
Why Local Hooks Aren't Shared by Default
Machine-specific Logic: Git hooks are often used to enforce rules or perform actions that are specific to the machine where they're executed. For example, a pre-commit hook might check for local environment variables like PYTHONPATH to ensure a Python script has the necessary dependencies.
User Customization: Developers can modify and customize their local hooks according to their project's needs without affecting other developers' projects.
How Husky Gets Around This
Global Hook Execution Context: Husky operates by creating symbolic links (symlinks) from global locations to custom hook files within the repository.
Environment Agnostic: By running these symlinks through a Node.js process, Husky abstracts away the machine-specific logic and ensures that hooks are executed in an environment-agnostic way.
Detailed Steps:

Install Husky Globally:

BASH
npm install --save-dev husky

Initialize Husky on Project:

BASH
npx husky-init
npm install

Create Custom Hooks: Place your custom hooks in the .husky directory.

Configure Git to Run Symlinks:

Husky generates symlinks from global locations (e.g., node_modules/.bin) to your project's hook files.

#### Closing Summary

Husky addresses the limitations of local hooks by leveraging symlinks and a Node.js environment, ensuring that custom hooks can be shared across developers while maintaining machine-specific logic in an abstracted manner. This tool effectively bridges the gap between local customization and global sharing.

#### Common Mistakes/Follow-ups
Misunderstanding Symlinks: Developers might think Husky creates actual copies of hook files rather than symlinks.
Environment Dependency: Not realizing that custom hooks can still depend on local environment variables, which Husky manages through its Node.js process.
Performance Overhead: Some may wonder about the performance impact of running hooks through a Node.js process; however, this overhead is generally negligible for typical Git operations.

**Code:**

```bash
# Install Husky globally and locally
npm install --save-dev husky

# Initialize Husky in your project
npx husky-init
npm install

# Create custom hooks
echo "#!/bin/sh" > .husky/pre-commit
```

**Flow:**

1. Install Husky
2. NPM Install
3. Husky Init
4. Create Custom Hooks
5. RUN Git Command with HuskySymlinks

