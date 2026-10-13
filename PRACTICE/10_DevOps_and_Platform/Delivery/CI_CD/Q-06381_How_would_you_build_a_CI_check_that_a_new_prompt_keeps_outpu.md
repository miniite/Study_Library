---
type: "practice"
difficulty: "Deep"
---

# How would you build a CI check that a new prompt keeps output in the required JSON schema the downstream service expects, given the model can drift into malformed output?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Data and Analytics CI Gates](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Data_and_Analytics_CI_Gates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a new AI-generated prompt maintains compliance with the expected JSON schema, we need to establish a robust CI check that validates each output against the schema before deployment. This involves several steps, including defining the schema, integrating validation logic into our pipeline, and setting up automated tests. Let's break down the process from first principles.

First, it’s crucial to understand that a schema defines the structure of the JSON data expected by the downstream service. For instance, if we expect an output in the form:

JSON
{
  "user_id": 12345,
  "message": "Hello, world!",
  "timestamp": "2023-10-01T12:00:00Z"
}

We need to ensure every generated JSON conforms to this structure.

**Code:**

```python
from jsonschema import validate, ValidationError

def validate_json_schema(json_data, schema):
    try:
        validate(instance=json_data, schema=schema)
    except ValidationError as e:
        return False, str(e)
    else:
        return True, "Valid JSON"
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "user_id": 12345,
  "message": "Hello, world!",
  "timestamp": "2023-10-01T12:00:00Z"
}

Output:

PLAINTEXT
Copy
Valid JSON

Input:

JSON
Copy
{
  "user_id": 12345,
  "message": "Hello, world!"
}

Output:

PLAINTEXT
Copy
ValidationError: ['timestamp' is a required property]
```

**Data:**

```json
{
  "schema": {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
      "user_id": {"type": "integer"},
      "message": {"type": "string"},
      "timestamp": {"type": "string", "format": "date-time"}
    },
    "required": ["user_id", "message", "timestamp"]
  }
}
```

