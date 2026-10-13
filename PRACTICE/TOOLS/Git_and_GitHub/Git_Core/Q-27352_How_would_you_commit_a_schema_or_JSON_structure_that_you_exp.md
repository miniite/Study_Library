---
type: "practice"
difficulty: "Intermediate"
---

# How would you commit a schema or JSON structure that you expect the LLM's output to follow, so a PR can enforce that the format didn't change?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Conventional Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Conventional_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When committing a schema or JSON structure that an LLM (Language Model) is expected to generate as part of your codebase, it's crucial to ensure consistency and correctness in the data formats across different outputs. This can be achieved by using Git hooks and GitHub Actions to enforce the format standards via pre-commit checks. Here’s how you could approach this:

First, you need to define a schema that describes the expected structure of the JSON output. For example, if your LLM generates configuration files or logs, you would create a JSON schema (.json file) that outlines all valid keys and their types.

#### Triaging the Question

The question is essentially asking how to ensure that any changes made by an LLM generate outputs in a consistent format without violating predefined structure rules. This involves setting up automated checks before committing code, ensuring that the PRs adhere to these schema definitions.

#### First-Principles Solution
Define the Schema: Create or use existing JSON schemas (using JSON Schema) that define the expected structure.
Set Up Pre-commit Hooks: Use Git hooks like pre-commit to run validation scripts before a commit is pushed.
GitHub Actions for CI/CD Integration: Integrate these checks into GitHub Actions to ensure they are run automatically when PRs are created.
Step-by-Step Solution

Define the JSON Schema:

JSON
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "LLM Output Schema",
  "type": "object",
  "properties": {
    "log_level": { "type": "string" },
    "timestamp": { "type": "integer" },
    "message": { "type": "string" }
  },
  "required": ["log_level", "timestamp", "message"]
}

Install a Validation Tool: Use jsonschema in Python to validate JSON against the schema.

Create Pre-commit Hook:

BASH
# .pre-commit-config.yaml
repos:
- repo: https://github.com/pre-commit/pre-commit-hooks
  hooks:
  - id: check-json

Write a Validation Script in Python (using jsonschema):

PYTHON
import jsonschema
import json

# Load the schema and JSON data from files
with open('schema.json', 'r') as f:
    schema = json.load(f)

with open('output.json', 'r') as f:
    data = json.load(f)

# Validate the JSON against the schema
try:
    jsonschema.validate(data, schema)
    print("JSON is valid.")
except jsonschema.exceptions.ValidationError as ve:
    print("Invalid JSON:", ve.message)

Configure GitHub Actions to run this script whenever a PR is opened or updated.

#### Closing Summary
and Common Mistakes

By setting up these checks, you ensure that any new output from the LLM adheres to the expected format without manual intervention. The key challenges include ensuring that the schema accurately represents all possible outputs and that the validation scripts handle edge cases correctly.

#### Common Mistakes
Incorrect Schema Definition: Failing to account for all necessary fields or incorrect data types.
Inadequate Error Handling: Not properly handling exceptions during JSON parsing or schema validation.
Over-reliance on Manual Checks: Relying solely on manual validation before committing can lead to inconsistencies.

