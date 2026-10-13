---
type: "practice"
difficulty: "Deep"
---

# Your agent's tool definitions (JSON schemas) live in Git and a schema PR merged that broke the model's tool-calling. How would a required contract-validation check on the PR have caught it before merge?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Automated Dependency Updates](<../../../../TOOLS/Git_and_GitHub/Git_Core/Automated_Dependency_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with AI models, ensuring robustness is crucial. The recent issue where a schema PR was merged and broke the model's tool-calling highlights the need for effective contract-validation checks in Git. In this context, we'll explore how a pre-merge validation check could have prevented such issues.

#### Triage
Understanding the Problem

The question asks about how a required contract-validation check during a Pull Request (PR) review process could have caught an issue before merging. This involves understanding the role of schemas in defining tool contracts and why validation checks are essential for maintaining model integrity.

Key Assumptions:
Schemas: The agent's tool definitions live in Git, and there is a schema defined to validate these definitions.
Validation Check: A pre-merge validation check ensures that the proposed changes adhere to the existing schemas.
Breakage: The PR broke the model’s ability to call tools correctly, indicating a mismatch between the current schema and the tool definitions.
First Principles: Designing Validation Checks

To address this issue effectively, we need to ensure that:

Schema Definition: A comprehensive schema is in place that captures all required fields.
Validation Process: An automated validation process runs before merging changes.
Tool Calling Logic: The model’s logic correctly calls tools based on the defined schemas.
Step-by-Step Solution
Define Schemas: Ensure a detailed JSON schema exists for tool definitions, capturing all necessary attributes and their constraints.
Automate Validation: Implement an automated validation script that runs during the PR process to check against the schema.
Integrate into Workflow: Integrate this validation as part of the CI/CD pipeline to ensure every change is validated before merging.

#### Example
Schema Definition
JSON
{
  "type": "object",
  "properties": {
    "name": { "type": "string" },
    "toolType": { "type": "string", "enum": ["API", "LocalScript"] },
    "requiredInputs": {
      "type": "array",
      "items": [
        { "type": "object", "properties": {"name": { "type": "string" }, "type": { "type": "string" }}}
      ]
    }
  },
  "required": ["name", "toolType", "requiredInputs"]
}
Validation Script

A simple validation script could look like this:

PYTHON
import json

def validate_schema(tool_def, schema):
    try:
        # Validate against the JSON schema using a library like jsonschema
        from jsonschema import validate, ValidationError
        with open('path_to_schema.json', 'r') as f:
            schema = json.load(f)
        validate(instance=tool_def, schema=schema)
        print("Schema validated successfully.")
    except ValidationError as ve:
        print(f"Validation error: {ve.message}")

# Example usage
tool_definition = {
    "name": "example_tool",
    "toolType": "API",
    "requiredInputs": [{"name": "input1", "type": "string"}]
}

schema_path = 'path_to_schema.json'
validate_schema(tool_definition, schema_path)

#### Closing Summary
and Common Mistakes

A comprehensive contract-validation check would have caught the issue by ensuring that any proposed changes to tool definitions are compatible with the existing schema. The key mistakes include:

Lack of Clear Schemas: Defining unclear or incomplete schemas.
Manual Validation: Relying on manual checks which can be error-prone and inconsistent.
Delayed Checks: Running validation only after a PR is merged, as in this case.
Common Follow-Ups
How would you design the schema to cover edge cases?
Can you explain how the validation script could be integrated into GitHub Actions or GitLab CI/CD?
What if there are multiple tools with complex interactions? How can validation handle such scenarios?

**Comparison:**

```
Schema Property	Type	Constraints
name	string	Non-empty
toolType	string	Enum: ['API', 'LocalScript']
requiredInputs	array	Array of objects, each with name and type
```

**Code:**

```python
import json

def validate_schema(tool_def, schema):
    try:
        from jsonschema import validate, ValidationError
        with open('path_to_schema.json', 'r') as f:
            schema = json.load(f)
        validate(instance=tool_def, schema=schema)
        print("Schema validated successfully.")
    except ValidationError as ve:
        print(f"Validation error: {ve.message}")

# Example usage
tool_definition = {
    "name": "example_tool",
    "toolType": "API",
    "requiredInputs": [{"name": "input1", "type": "string"}]
}

schema_path = 'path_to_schema.json'
validate_schema(tool_definition, schema_path)
```

**Example input / output:**

```text
# Example input for validation
tool_definition = {
    "name": "example_tool",
    "toolType": "API",
    "requiredInputs": [{"name": "input1", "type": "string"}]
}

schema_path = 'path_to_schema.json'

# Validate the tool definition
validate_schema(tool_definition, schema_path)
```

**Data:**

```json
{
  "name": "example_tool",
  "toolType": "API",
  "requiredInputs": [{"name": "input1", "type": "string"}]
}
```

**Flow:**

1. Tool Definition PR
2. Run Validation?
3. Passes Validation
4. Merge PR
5. Failed Validation
6. Fix Issues, Re-run Validation

