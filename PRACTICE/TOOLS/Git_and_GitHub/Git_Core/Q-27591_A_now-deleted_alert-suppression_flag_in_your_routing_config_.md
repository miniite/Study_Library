---
type: "practice"
difficulty: "Deep"
---

# A now-deleted alert-suppression flag in your routing config caused a real page to be dropped, and you must find when it was added and later removed across tens of thousands of commits. Explain git log -S versus -G for this hunt and why literal-versus-regex matters when the flag name overlaps other config keys.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve the problem of finding a deleted alert-suppression flag in your routing configuration across tens of thousands of commits, we need to leverage Git's git log command with different flags. Specifically, we'll use -S and -G. The core idea is that these commands help us search for insertions (additions) and deletions of specific text patterns within the commit history.

#### Triage
Understanding the Question

The question is asking you to find a deleted flag by searching through thousands of commits. This involves understanding how Git logs changes and using the appropriate flags to perform an effective search. The key is recognizing that literal versus regex matters due to potential overlaps in configuration keys.

#### First-Principles Solution
Identify the Flag Name: Let's assume the deleted flag name is alert.suppression_flag. This name will be used as our search pattern.
Using -S and -G:
git log -S <pattern>: Searches for lines that were inserted or deleted matching a given hunk of text (a sequence of changes). In this case, it can help find the specific change where the flag was added.
git log -G <pattern>: Searches for lines containing the given regular expression. This is useful when you are searching for the deletion of a line that matches the pattern.
Detailed Steps
Identify the Flag Name and Pattern:
The flag name is alert.suppression_flag.
Using -S to Find Insertion:
SH
git log -S "alert.suppression_flag"
This will show all commits where this string was inserted or modified in a way that could indicate the addition of the flag.
Using -G to Find Deletion:
SH
git log -G "^alert\.suppression_flag$"
The regex pattern ^alert.suppression_flag$ ensures we are looking for lines exactly matching the flag name, which will help us find where it was removed.

#### Closing Summary
and Common Mistakes/Follow-Ups

Using -S and -G with appropriate patterns is a powerful way to narrow down the search. For -S, you look for insertions or modifications that might add the flag, while -G helps identify deletions by ensuring exact matches. The key insight here is recognizing that using regex can help avoid false positives from other config keys.

#### Common Mistakes
and Follow-Ups
Literal vs Regex: Using alert.suppression_flag without quotes can cause issues if there are spaces or special characters in the flag name.
Performance Consideration: Searching through tens of thousands of commits can be time-consuming. Ensure you have an efficient setup, such as a Git repository with proper reflog and history pruning.
Edge Cases: What if the flag was modified but not deleted? In such cases, -S might help find where it was added or modified.

**Code:**

```
SH
Copy
git log -S "alert.suppression_flag"
git log -G "^alert\.suppression_flag$"
```

**Example input / output:**

```

Example command output:

PLAINTEXT
Copy
commit 1234567890abcdef (HEAD -> main)
Author: John Doe <john.doe@example.com>
Date:   Mon Jan 1 12:00:00 2024 +0000

    Added alert-suppression_flag for production

diff --git a/routing/config.js b/routing/config.js
--- a/routing/config.js
+++ b/routing/config.js
@@ -10,6 +10,7 @@ const config = {
     "flag1": true,
     "flag2": false,
+    "alert.suppression_flag": true,

commit abcdef1234567890 (origin/master)
Author: Jane Doe <jane.doe@example.com>
Date:   Mon Jan 15 12:00:00 2024 +0000

    Removed alert-suppression_flag after review

diff --git a/routing/config.js b/routing/config.js
--- a/routing/config.js
+++ b/routing/config.js
@@ -10,7 +10,6 @@ const config = {
     "flag1": true,
     "flag2": false,

-    "alert.suppression_flag": true,
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[Identify Flag Name] --> B{Use -S or -G?};
4. B --> |Insertion| C[git log -S "alert.suppression_flag"];
5. B --> |Deletion| D[git log -G "^alert\.suppression_flag$"];
6. C --> E[Check Commits];
7. D --> F[Check Commits];
8. E --> G[Identify Add Commit];
9. F --> H[Identify Delete Commit];
10. G --> I[Mark Alert Suppression Flag Added];
11. H --> J[Mark Alert Suppression Flag Removed];
12. I --> J;

