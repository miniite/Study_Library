---
type: "practice"
difficulty: "Intermediate"
---

# You want the LLM to always return valid JSON. How would you use Copilot to enforce a structured output schema and validate it after the call?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Prompting for Validation and Explanations](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Validation_and_Explanations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that Copilot always returns valid JSON, we need to design an approach that enforces a structured output schema and validates the response post-generation. This involves defining clear schemas upfront, integrating them with Copilot's responses, and setting up validation mechanisms. By doing so, we can maintain consistency and reliability in our AI-assisted coding environment.

The key idea is to use JSON schemas to define the expected structure of the JSON output. We will integrate these schemas with Copilot through a middleware layer that preprocesses user inputs and postprocesses Copilot outputs. This ensures that the generated code aligns with our defined schema, thereby producing valid JSON every time.

**Code:**

```python
from jsonschema import validate
import json

# Define the JSON schema for our structured output
output_schema = {
    "type": "object",
    "properties": {
        "id": {"type": "integer"},
        "name": {"type": "string"},
        "data": {"type": "array", "items": {"type": "object"}}
    },
    "required": ["id", "name"]
}

def validate_json(response_text):
    try:
        response_dict = json.loads(response_text)
        validate(instance=response_dict, schema=output_schema)
        return True
    except (json.JSONDecodeError, ValidationError) as e:
        print(f"Invalid JSON: {str(e)}")
        return False

# Example Copilot call and validation function usage
copilot_response = """
{
  "id": 123,
  "name": "example",
  "data": [
    {"key": "value"},
    {"key2": "value2"}
  ]
}
"""

if validate_json(copilot_response):
    print("Valid JSON")
else:
    print("Invalid JSON, correcting...")
```

**Example input / output:**

```

{ "id": 123, "name": "example", "data": [ {"key": "value"}, {"key2": "value2"} ] }

CODE
Copy

After validation, the response is:
```plaintext
Valid JSON
```

**Flow:**

1. User Request
2. Copilot Response
3. Validation Middleware

