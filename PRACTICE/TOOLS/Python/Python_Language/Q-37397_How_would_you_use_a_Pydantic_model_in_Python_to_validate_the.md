---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a Pydantic model in Python to validate the structured JSON you asked an LLM to produce?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To validate structured JSON produced by an LLM, we can use the pydantic library in Python. It allows us to define data models and automatically validates any incoming data against those models. This is particularly useful when dealing with external data sources like AI-generated outputs.

#### Triage

The core of this question revolves around understanding how to integrate pydantic validation into a pipeline where an LLM generates JSON data, ensuring that the generated data conforms to expected schemas before further processing or storage. The interviewer might be probing your knowledge of:

How to define Pydantic models.
Validating incoming JSON against these models.
Handling potential validation errors.
Integrating this validation step into a broader workflow.

#### First-Principles Solution
Define the Model: Create a Pydantic model that matches the expected structure of the LLM-generated JSON.
Validate Data: Use this model to validate the incoming JSON data from the LLM.
Handle Errors Gracefully: Implement error handling to manage cases where the JSON does not conform to the schema.

#### Example

Let's assume we have an LLM that generates a response in the form of structured JSON, like so:

JSON
{
  "name": "Alice",
  "age": 25,
  "email": "alice@example.com"
}

We want to ensure this data conforms to the expected structure. Here's how we can set up our validation:

Step 1: Define the Model

First, install pydantic if you haven't already:

BASH
pip install pydantic

Next, define a Pydantic model for the JSON structure:

PYTHON
from typing import Optional
from pydantic import BaseModel, EmailStr

class Person(BaseModel):
    name: str
    age: int
    email: Optional[EmailStr] = None  # Using Optional to handle absence of this field if needed
Step 2: Validate Data

Now, let's create a function that takes the JSON data from an LLM and validates it using our Person model:

PYTHON
import json
from pydantic import ValidationError

def validate_person_data(json_str):
    try:
        # Parse the JSON string to a Python dictionary
        person_dict = json.loads(json_str)

        # Validate against the Pydantic model
        person = Person(**person_dict)
        print("Validation successful!")
        return person
    except ValidationError as e:
        print(f"Validation error: {e}")
        return None

# Example usage
llm_json_output = '{"name": "Alice", "age": 25, "email": "alice@example.com"}'
validated_person = validate_person_data(llm_json_output)
print(validated_person)  # Output the validated data or handle appropriately
Step 3: Handling Errors Gracefully

In the validate_person_data function, we catch and print any validation errors. You might want to log these errors or handle them more gracefully depending on your application's requirements.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By defining a Pydantic model and using it to validate JSON data from an LLM, you can ensure that the generated data is structured correctly before further processing. Some common mistakes include:

Not handling missing fields properly.
Overlooking validation errors, which could lead to silent failures.

#### Follow-up
questions might involve discussing how to scale this approach for multiple models or integrating it with a larger system.

