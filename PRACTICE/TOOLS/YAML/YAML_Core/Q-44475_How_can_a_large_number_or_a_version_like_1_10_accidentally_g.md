---
type: "practice"
difficulty: "Deep"
---

# How can a large number or a version like 1.10 accidentally get misread in YAML? How do you prevent it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with YAML, a common pitfall is how it handles numbers and strings. In YAML, a number that looks similar to a version string can be accidentally parsed as an integer or float, leading to unexpected behavior in your configuration files. For instance, the version 1.10 might get misinterpreted as 1.1, which could cause significant issues if you expect it to be treated as 1.10.

To prevent such misinterpretations, we need to ensure that our YAML values are explicitly marked as strings when necessary. This can be achieved using quotes around the value. Here’s a step-by-step guide on how to handle this situation.

**Code:**

```yaml
# Incorrect version in YAML (interpreted as 1.1 instead of 1.10)
version: 1.10

# Corrected version with explicit string formatting
version: "1.10"
```

**Example input / output:**

```text
# Incorrect input:
- version: 1.10

# Correct input:
- version: "1.10"
```

