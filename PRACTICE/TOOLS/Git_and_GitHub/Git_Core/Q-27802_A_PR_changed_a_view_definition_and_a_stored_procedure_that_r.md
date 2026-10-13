---
type: "practice"
difficulty: "Intermediate"
---

# A PR changed a view definition and a stored procedure that reads it in the same commit, and a reviewer can't tell whether a broken query came from the view or the proc. How would you split concerns across commits so a bad result is bisectable?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a PR changes both a view definition and a stored procedure that reads it, making it hard to trace broken queries back to the correct source, we need to ensure that each major change is contained within its own commit. This way, if something breaks, we can easily bisect the history to find the root cause. Here's a step-by-step approach:

Identify the Changes: First, identify which part of the codebase was modified—specifically, whether it was the view or the stored procedure.
Refactor the Code: Refactor the changes so that each significant alteration is in its own commit.
Add Tests: Ensure you have comprehensive tests to verify both the view and the stored procedures before committing.

By following these steps, we can maintain a clear history of changes, making it easier to debug issues if they arise.

**Code:**

```
DIFF
Copy
- # Initial PR (broken)
+ # Refactored PR

# Original code in views/view1.sql
CREATE OR REPLACE VIEW view1 AS
SELECT column1, column2 FROM table1;

# New code with separate commits:
# Step 1: Change the view definition
diff --git a/views/view1.sql b/views/view1.sql
new file mode 100644
index 0000000..f3b9c78
--- /dev/null
+++ b/views/view1.sql
@@ -0,0 +1,2 @@
+CREATE OR REPLACE VIEW view1 AS
+SELECT column1, column2 FROM table1;

# Step 2: Update the stored procedure to read from the new view
diff --git a/stored_procedures/proc1.sql b/stored_procedures/proc1.sql
index f3b9c78..0a68f4e 100644
--- a/stored_procedures/proc1.sql
+++ b/stored_procedures/proc1.sql
@@ -1,2 +1,3 @@
 CALL proc1();
+SELECT * FROM view1;
```

**Flow:**

1. Yes
2. No
3. Start
4. Identify changes
5. Is view change?
6. Create new commit for view
7. Create new commit for storedprocedure
8. Update tests and run
9. Update tests and run
10. Commit changes
11. Bisect history if needed
12. Fix issues or merge as is

