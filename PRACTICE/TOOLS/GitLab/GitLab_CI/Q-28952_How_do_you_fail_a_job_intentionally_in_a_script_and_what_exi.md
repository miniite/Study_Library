---
type: "practice"
difficulty: "Basic"
---

# How do you fail a job intentionally in a script and what exit code does GitLab check?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Debugging Runner Failures](<../../../../TOOLS/GitLab/GitLab_CI/Debugging_Runner_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitLab CI, failing a job intentionally in a script is a common practice to control pipeline execution flow or trigger specific actions. The key point here is understanding how GitLab interprets exit codes and uses them as indicators of job success or failure.

#### Triaging the Question

GitLab CI scripts use standard exit codes (0 for success, non-zero for failures) to inform the pipeline about the outcome of a job. To fail a job intentionally in your script, you can set an exit code that GitLab will recognize. The exact exit code and the mechanisms used by GitLab to check it are what we need to focus on.

#### First-Principles Solution

In a shell script, setting exit 1 or any non-zero value will make GitLab mark the job as failed. Conversely, an exit code of 0 signifies success. Here’s how you can intentionally fail a job:

BASH
#!/bin/bash

# Your script logic here...

# Intentionally failing the job
exit 1

#### Closing Summary
and Common Mistakes/Follow-Ups

To intentionally fail a GitLab CI job, use exit with a non-zero code. Common mistakes include not setting any exit value, which defaults to success (0), or using zero by mistake when you want the job to fail.

A follow-up question might be how to handle different failure scenarios and what other strategies can be used in GitLab CI to control job execution based on these exit codes.

