---
type: "practice"
difficulty: "Intermediate"
---

# How would git blame help you find who last changed the latency alert threshold and when, during a false-alarm investigation?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Blame and Ignore Revisions](<../../../../TOOLS/Git_and_GitHub/Git_Core/Blame_and_Ignore_Revisions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to understand how git blame can be used in conjunction with Git repositories hosted on GitHub or similar platforms to identify who last modified a specific line of code and when, particularly during an investigation into latency alert thresholds that might have caused false alarms. This tool is essential for AIOps engineers to quickly pinpoint the origin of changes in critical system configurations.

First, let's break down what git blame does: it attaches the commit hash to each line of a file, showing which revision was responsible for each line. By examining these revisions, we can trace back who last modified a specific part of the code and when.

For example, if you're investigating why a particular latency alert threshold started firing incorrectly, you would use git blame on the relevant configuration file (e.g., alert-latency-config.yaml) to identify which line was last changed. Here’s how we can approach this step-by-step:

Identify the Specific Line: First, locate the specific line of code that defines the latency threshold in your repository.
Run git blame: Execute a command like git blame --line-porcelain path/to/config.yaml | grep "line_number" to pinpoint which commit and author last modified this line.
Analyze the Output: The output will show you the commit hash, author name, and date for each line. For example: @@ -1 +1 @@ 8f92b4e11 30 minutes ago | Author: John Doe.

Now let's walk through an example scenario:

#### Triaging the Question

The question is asking how to use Git tools, specifically git blame, to trace back a change in the latency alert threshold configuration. The interviewer might be probing your understanding of version control systems, your ability to effectively search and navigate code history, and your problem-solving skills.

#### First-Principles Solution

Identify the File: Ensure you have the correct file path, e.g., alert-latency-config.yaml.

Run git blame:

BASH
git blame --line-porcelain alert-latency-config.yaml | grep "line_number"

For example, if we want to find out who changed line 10, the command would be:

BASH
git blame --line-porcelain alert-latency-config.yaml | grep '^[^ ]* [0-9] *10'

Analyze the Output:

CODE
@@ -1 +1 @@
8f92b4e11 30 minutes ago | Author: John Doe

This output shows that line 10 was last modified by John Doe with the commit hash 8f92b4e11, which occurred 30 minutes ago.

#### Closing Summary
and Common Mistakes/Follow-Ups

By using git blame, we can efficiently trace back changes in critical configuration files, aiding in investigations of false alarms. However, common mistakes to avoid include not specifying the exact line number or file path correctly, leading to incorrect results.

Possible follow-up questions could involve discussing best practices for version control and configuration management, such as creating feature branches before making significant changes or using code reviews to ensure quality.

**Code:**

```bash
git blame --line-porcelain alert-latency-config.yaml | grep '^[^ ]* [0-9] *10'
```

**Example input / output:**

```text
@@ -1 +1 @@
8f92b4e11 30 minutes ago | Author: John Doe
```

