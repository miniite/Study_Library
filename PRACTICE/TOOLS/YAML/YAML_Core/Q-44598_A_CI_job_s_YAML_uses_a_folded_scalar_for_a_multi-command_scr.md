---
type: "practice"
difficulty: "Intermediate"
---

# A CI job's YAML uses a folded scalar for a multi-command script and lines got joined wrong, breaking the shell. Why did folding collapse the newlines and how would you fix it?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with CI/CD pipelines using YAML, it's crucial to understand the nuances of different scalar types, especially when dealing with multi-line scripts or commands. In this case, the issue arises from the use of a folded scalar in YAML, which can cause newlines to be collapsed during parsing, thus breaking the shell script.

Triaging the Issue

The question revolves around why a folded scalar in YAML caused newlines to be joined incorrectly and how to fix it. A folded scalar in YAML is denoted by preceding the value with > and typically represents multiline text where leading whitespaces are significant but trailing whitespace is ignored. However, this behavior can sometimes lead to unintended consequences if not managed properly.

#### First-Principles Solution

To address the issue, we need to ensure that each command in the script remains on a separate line within the YAML file. This can be achieved by either using block scalars (| for literal multiline strings or > for folded multiline strings) correctly or by carefully formatting the script.

Step-by-Step Solution
Identify the Issue: The folded scalar is causing the newlines to be collapsed because it interprets leading spaces as significant.
Rewrite the Script:
Use a block scalar (| for literal or > for folded) instead of a folded scalar.
Ensure each command in the shell script occupies its own line.

Here’s an example YAML snippet:

YAML
- run: |
    echo "Starting first command"
    sleep 5
    echo "Finishing first command"
    echo "Starting second command"
    sleep 10
    echo "Finishing second command"

In this reformatted script, each echo and sleep command is on a separate line, ensuring that the shell interprets them correctly as distinct commands.

#### Common Mistakes
and Follow-Ups
Not Distinguishing Between Scalar Types: Forgetting to differentiate between folded (>) and block (|) scalars can lead to issues like this.
Ignoring Leading Whitespace: Folded scalars treat leading whitespace as significant, so ensuring proper formatting is crucial.
Lack of Testing: Always test the YAML configuration in a safe environment before deploying it to production.

#### Closing Summary

By understanding the behavior of different scalar types in YAML and carefully structuring multi-line scripts, you can avoid such issues. Properly formatted YAML ensures that shell commands are interpreted correctly by the CI/CD system.

**Code:**

```yaml
- run: |
    echo "Starting first command"
    sleep 5
    echo "Finishing first command"
    echo "Starting second command"
    sleep 10
    echo "Finishing second command"
```

**Example input / output:**

```text
No input-output examples are necessary for this solution, as it focuses on the YAML syntax and formatting.
```

