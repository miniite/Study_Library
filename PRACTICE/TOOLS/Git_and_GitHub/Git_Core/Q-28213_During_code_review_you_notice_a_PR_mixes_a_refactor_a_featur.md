---
type: "practice"
difficulty: "Intermediate"
---

# During code review you notice a PR mixes a refactor, a feature, and a bugfix in one commit. Why does this matter for revertability and bisect, and how would you coach the author to restructure it?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Merging multiple types of changes (refactor, feature, bugfix) into a single commit can significantly impair revertability and bisecting capabilities in Git. To improve maintainability, we should separate these changes into distinct commits. This ensures that when issues arise, or if a change needs to be rolled back, each component is isolated, making the process simpler and more controlled.

#### Triage

The question asks about the implications of mixing different types of changes (refactor, feature, bugfix) within a single commit during code review. Specifically, it focuses on how such practices affect revertability and bisecting capabilities. The interviewer is likely probing your understanding of Git best practices, version control principles, and refactoring techniques.

#### First-Principles Solution
Understand the Changes: Identify the different types of changes in the commit—refactor, feature, and bugfix.
Separate into Distinct Commits: Split each change into its own commit to maintain clarity and allow for targeted reverts if necessary.
Documentation: Ensure that comments or documentation are added within the code to explain why a particular refactor was made or how the new feature impacts existing functionality.
Step-by-Step Solution

Identify Changes:

Refactor: Improvements in code structure, such as renaming functions or refactoring logic.
Feature: New functionality that extends the product's capabilities.
Bugfix: Fixes for known issues or bugs in the current codebase.

Separate Commits:

Create a new commit for the refactor changes.
DIFF
@@ -10,6 +10,7 @@
  function oldFunction() {
      // original logic
  }
CODE
 function newFunction() {
     // refactored logic
 }
CODE
- Add a comment explaining why this refactoring was necessary: `// Refactor to improve readability and maintainability of existing code`.

Feature Commit:

Create another commit for the feature.
JAVASCRIPT
function newFeature() {
    // new functionality implementation
}
Add comments describing how the new feature integrates with existing systems or any dependencies it has.

Bugfix Commit:

Finally, create a separate commit for the bugfix.
DIFF
@@ -20,7 +20,8 @@
    // previous logic
-    console.log("old state");
     // updated logic to handle new state correctly
CODE
   if (condition) {
CODE

#### cons
ole.log("new state");
CODE
   }
CODE
- Comment on the bug and how it was resolved: `// Fixed a bug in handling edge cases`.

Communication:
Coach the author on why separating these changes is beneficial.
Discuss best practices for version control and commit messages.

#### Closing Summary

By separating refactoring, features, and bugfixes into distinct commits, we improve code revertability and bisecting capabilities. This allows us to pinpoint exactly which change caused an issue if needed, making debugging more efficient. Additionally, clear commit messages help maintain a well-organized history that future developers can easily navigate.

#### Common Mistakes
and Follow-Ups
Not Understanding the Changes: Sometimes, developers may not fully understand the impact of their changes, leading to poorly named or unrelated commits.
Ignoring Best Practices: Developers might be tempted to simplify by combining multiple changes for convenience, but this can introduce maintenance issues down the line.

To address these concerns:

Ensure code reviews focus on both functionality and commit structure.
Use tools like GitKraken or SourceTree to visualize commit history, making it easier to spot issues early.

