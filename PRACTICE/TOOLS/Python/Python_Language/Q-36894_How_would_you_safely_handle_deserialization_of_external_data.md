---
type: "practice"
difficulty: "Deep"
---

# How would you safely handle deserialization of external data in Python so that a malicious payload can't trigger code execution or resource exhaustion (billion laughs, deep nesting)?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prompt Injection Defense](<../../../../TOOLS/Python/Python_Language/Prompt_Injection_Defense.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with deserialization of external data in Python, ensuring safety is crucial to prevent potential security vulnerabilities like code injection and resource exhaustion attacks, such as the "Billion Laughs" attack or excessive deep nesting issues. This problem requires a comprehensive approach that involves understanding the risks, selecting appropriate libraries, validating inputs, and implementing robust defensive measures.

#### Triage

The question asks how to safely handle deserialization of external data in Python. The interviewer is probing for an understanding of potential security threats associated with deserialization, such as code injection or resource exhaustion. Key assumptions include that you are using a specific external library for deserialization (e.g., pickle, json) and that the input data can come from untrusted sources.

#### First-Principles Solution
Use Safe Libraries: Avoid libraries like pickle which allow full code execution, and use safer alternatives like json.
Input Validation: Validate the structure and content of the incoming data to ensure it conforms to expected patterns.
Limit Resource Usage: Implement controls to prevent excessive resource consumption (e.g., deep nesting).
Exception Handling: Catch potential exceptions during deserialization and handle them gracefully.
Step-by-Step Solution

Choosing a Safe Library:

Prefer json over pickle because it is safer for untrusted data.
PYTHON
import json

Input Validation:

Define a schema or structure that the deserialized data should adhere to.
Use libraries like pydantic to enforce validation rules.
PYTHON
from pydantic import BaseModel

class MyDataModel(BaseModel):
    name: str
    nested_data: dict = {}

Limiting Resource Usage:

Set limits on the maximum depth of nested structures using a custom deserialization function.
PYTHON
def safe_deserialize(data, max_depth=5):
    try:
        # Custom logic to check and limit nesting
        return json.loads(data)
    except (ValueError, RecursionError) as e:
        raise ValueError("Invalid or deeply nested data")

# Example usage
valid_data = '{"name": "John", "nested_data": {}}'
invalid_data = '{"name": "John", "nested_data": {"nested_data": {"nested_data": {"nested_data": ...}}}}'  # Deeply nested
try:
    safe_deserialize(valid_data)
except ValueError as e:
    print(e)  # Handle valid data

try:
    safe_deserialize(invalid_data)
except ValueError as e:
    print(e)  # Handle invalid, deeply nested data

Exception Handling:

Catch exceptions during deserialization and handle them appropriately.
PYTHON
try:
    json.loads(data)
except (ValueError, RecursionError):
    # Log the error or take appropriate action
    print("Deserialization failed due to invalid data")

#### Closing Summary

To safely handle deserialization in Python, it is essential to use safe libraries like json, validate input structures, limit resource usage through custom functions, and implement robust exception handling. This approach ensures that malicious payloads are neutralized before they can cause harm or exhaust system resources.

#### Common Mistakes/Follow-Ups
Misusing Unsafe Libraries: Continuing to use pickle for untrusted data.
Lack of Input Validation: Not defining clear schema rules for expected data structures.
Overlooking Resource Limits: Failing to implement checks for deep nesting or other resource-heavy scenarios.

