---
type: "practice"
difficulty: "Basic"
---

# How would you use git diff to review exactly which lines of a stored procedure you changed before committing it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Reviewing Changes with Diff](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reviewing_Changes_with_Diff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To effectively use git diff for reviewing changes in a stored procedure before committing, we need to understand how Git works and what git diff can show us. This command compares differences between two versions of files, which is perfect for spotting the specific lines that have been modified.

Firstly, ensure you are working from the correct branch where your changes were made (typically feature or development branches). Then, after committing locally with a descriptive commit message, use git diff to see what changes were introduced. This process involves several steps: understanding the command syntax, using it effectively, and interpreting its output.

**Code:**

```
SH
Copy
# Example usage of git diff before committing
git diff HEAD~1
```

**Example input / output:**

```text
diff --git a/stored_procedures/my_proc.sql b/stored_procedures/my_proc.sql
index 5f0d9e4..2b81f73 100644
--- a/stored_procedures/my_proc.sql
+++ b/stored_procedures/my_proc.sql
@@ -4,7 +4,7 @@
     -- Initial procedure body
-    IF condition THEN
+    IF new_condition THEN
         -- Updated logic here
     END IF;

@@ -10,5 +10,6 @@
     -- Additional code
 END;
 $$ LANGUAGE plpgsql;
```

**Flow:**

1. Open Terminal
2. Use `git diff` with a referencecommit
3. Run `git diff HEAD~1` in thestored procedure directory
4. Review the output for linedifferences
5. Commit changes with anappropriate message ifsatisfied

