---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git bisect with a simple check to find the commit where a recording rule started returning no data?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Bisect](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Bisect.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with issues in monitoring and alerting systems, such as a recording rule suddenly not returning any data, Git Bisect is an invaluable tool that helps pinpoint exactly when this change occurred. This method essentially narrows down the problematic commit by repeatedly dividing the history into two halves until the offending commit is isolated.

#### Triaging the Question

This question asks how to use git bisect with a simple check to find the commit where a recording rule in an AIOps system started returning no data. The key here is that you need to set up a script or command that can determine whether a specific version of the codebase (or more specifically, a particular commit) has this issue. git bisect will then use binary search to find the exact commit.

#### Assumptions
and Setup
You have access to the Git repository containing your AIOps monitoring system.
You know how to run the application or tests in different versions of the codebase.
The recording rule is part of a running service that can be checked by some command-line tool or script.
Step-by-Step Solution Using First Principles

Identify the Problem Range:

Determine the range of commits between when the monitoring was working correctly and when it stopped returning data. This could be based on known changes to the recording rule or a specific date range.

Run git bisect start:

Begin by initializing Git Bisect:
SH
git bisect start <good-commit> <bad-commit>

Here, <good-commit> is the commit where everything was working fine and <bad-commit> is a recent commit you know introduced issues.

Check the Current Version:

Run your check (e.g., curl or a script) against the current version of the codebase to see if it's returning data as expected.
SH
git bisect bad # If the current state does not return any data

Switch and Check Again:

Switch to the previous commit by running git bisect good or git bisect bad, depending on the outcome of your check.
Repeat this process, switching commits and checking the state until you narrow down to a single commit.

Scripting the Check:

Write a simple script that can be run from the command line to check the recording rule status. For example:
SH
#!/bin/bash
./check_recording_rule.sh
if [ $? -ne 0 ]; then
    echo "No data returned."
    exit 1
fi
echo "Data is returning normally."

Binary Search:

Run git bisect repeatedly, each time providing feedback on whether the check passed or failed.
SH
git bisect run /path/to/check_script.sh
This will automatically switch between commits and run your script until it isolates the problematic commit.

Identify the Problem Commit:

Once git bisect finishes, the last command will indicate that you've found the bad commit.
SH
git bisect good # Indicate everything is fine from this point onward

Final Steps:

Review the log for the problematic commit to understand what changed.
Document the issue and its cause.

#### Closing Summary

Using git bisect efficiently narrows down the exact commit that introduced a bug or change, making it easier to debug and resolve issues in AIOps systems. By setting up your checks properly and using Git Bisect’s binary search capabilities, you can quickly pinpoint the problematic code change without manually testing every single commit.

#### Common Mistakes/Follow-Ups
Not Setting Good/Bad Correctly: Always ensure that git bisect knows which versions are good or bad. This is crucial for the algorithm to work correctly.
Improper Scripting: Ensure your check script accurately reflects the state you’re looking for. A flaky or inconsistent check can lead to misleading results.
Not Documenting: Once the problematic commit is identified, document it thoroughly so that others can understand and replicate your findings.

