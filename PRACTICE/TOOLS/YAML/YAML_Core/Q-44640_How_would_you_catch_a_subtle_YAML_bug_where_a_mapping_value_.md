---
type: "practice"
difficulty: "Intermediate"
---

# How would you catch a subtle YAML bug where a mapping value spanning multiple lines without a block scalar silently folded into the next key? Give the detection approach.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Quoting and Scalars](<../../../../TOOLS/YAML/YAML_Core/Quoting_and_Scalars.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with YAML, it's crucial to be aware of how mappings and scalars are interpreted, especially when they span multiple lines. In this scenario, we need to detect a subtle bug where a mapping value that spans multiple lines is silently folded into the next key instead of being treated as a separate scalar. This can lead to unexpected behavior in our configuration files or deployment scripts.

#### Triage
The question is asking us to identify and prevent a YAML parsing issue where a value spanning multiple lines is incorrectly interpreted by the parser, causing it to be merged with the subsequent key. This requires understanding how YAML handles multi-line strings (block scalars) versus regular scalar values. We must ensure that our YAML configurations are correctly formatted to avoid such issues.

First-principles approach: Let's start by understanding the problem in detail. When a mapping value spans multiple lines, it should ideally be treated as a separate scalar unless explicitly indicated otherwise using block style. However, if not handled properly, many parsers might concatenate these values into the next key, leading to incorrect configuration or deployment.

Understand YAML Syntax: YAML distinguishes between flow-style and block-style scalars. Flow-style is used for simple strings (single-line) while block-style uses indentation to denote scalar values that span multiple lines.
Detecting Misplaced Indentation: The key issue here is ensuring that any multi-line value is correctly defined using the | or > indicators to indicate a block scalar. Without these, the parser might concatenate the value with the next key.
Code Review and Linters: Implement code review practices and use YAML linters like yamllint to automatically detect such issues during development.

To illustrate this, let's consider an example where we expect value1 and value2 to be separate scalar values but are incorrectly concatenated:

YAML
# Incorrect YAML configuration
foo: value1
   - bar:
       baz: value2

In the above example, if not handled correctly, value1 might be concatenated with - bar: resulting in an unexpected mapping. This can be fixed by properly defining the block scalar for value2:

YAML
# Correct YAML configuration
foo: value1
- bar:
    baz: |-
      value2

Here, |- is used to indicate a block scalar that spans multiple lines.

#### Closing summary
To avoid such subtleties in YAML, always ensure that multi-line values are correctly identified using block scalars. Utilizing YAML linters and implementing code reviews can help catch these issues early. Common mistakes include missing block scalar indicators or incorrect indentation, leading to silent concatenations of key-value pairs.

**Code:**

```yaml
# Incorrect YAML configuration
foo: value1
   - bar:
       baz: value2

# Corrected YAML configuration
foo: value1
- bar:
    baz: |-
      value2
```

**Flow:**

1. Yes
2. No
3. Load YAML File
4. Parse YAML
5. Is Block Scalar CorrectlyIndicated?
6. Proceed with Parsing
7. Linter Error Detected

