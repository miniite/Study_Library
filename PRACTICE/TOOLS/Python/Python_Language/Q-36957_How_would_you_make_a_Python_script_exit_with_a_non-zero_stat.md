---
type: "practice"
difficulty: "Basic"
---

# How would you make a Python script exit with a non-zero status code so a CI pipeline detects that a step failed?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Graceful Shutdown](<../../../../TOOLS/Python/Python_Language/Graceful_Shutdown.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a CI pipeline recognizes that a Python script has failed, we need to make the script exit with a non-zero status code. This is crucial because most CI systems check the return value of commands executed in the pipeline and use it to determine if a step succeeded or failed. By default, a Python script exits with a zero status code (indicating success) when it completes execution without errors. To change this behavior, we need to explicitly exit with a non-zero code based on certain conditions.

