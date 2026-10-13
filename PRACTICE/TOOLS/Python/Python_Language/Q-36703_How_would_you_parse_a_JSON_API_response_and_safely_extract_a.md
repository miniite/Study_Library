---
type: "practice"
difficulty: "Intermediate"
---

# How would you parse a JSON API response and safely extract a nested field that might be missing?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To parse a JSON API response and safely extract a nested field, you need to handle potential missing fields gracefully to avoid runtime errors. This task involves understanding how Python’s dictionary operations work and leveraging exception handling techniques. The core idea is to traverse the JSON structure using a try-except block or conditional checks.

#### Triage

The question asks for a method to safely extract a nested field from a JSON API response, which might be missing. The interviewer is probing your knowledge of handling nested dictionaries in Python and error management practices.

#### Assumptions
The JSON response will be valid.
You are working with a dictionary structure that represents the JSON data.
The nested field path can vary (e.g., user.address.city).

#### Solution

Let's break down the problem step by step:

Understand the Structure: Identify how to navigate the nested fields using key paths.
Safe Extraction: Use Python’s dictionary operations and exception handling to safely access nested fields.
Step-by-Step Solution

Here is a function that takes a JSON response as a string, a field path (as a list of keys), and returns the value if found, or None otherwise:

PYTHON
def safe_extract(json_response: str, field_path: list) -> any:
    # Load JSON into a Python dictionary
    data = json.loads(json_response)

    current_dict = data

    try:
        for key in field_path:
            current_dict = current_dict[key]
    except (KeyError, TypeError):
        return None

    return current_dict

This function uses json.loads to convert the JSON string into a dictionary. It then iterates over each key in the provided path and navigates down the nested structure. If any of the keys are missing or if an invalid type is encountered, it catches the exception and returns None.

#### Closing Summary

The solution involves converting the JSON response to a Python dictionary and safely navigating through potential nested structures using a try-except block to catch key errors gracefully.

#### Common Mistakes
and Follow-Ups
Mistake: Not handling exceptions properly, leading to unhandled KeyError or TypeError.

#### Follow-Up
Discussing performance implications of deeply nested paths versus direct access.
Enhancements: Exploring alternative methods like using a recursive function for dynamic path handling.

**Comparison:**

```
Field Path	Output
["user", "address", "city"]	New York
["user", "state", "city"]	None
```

**Code:**

```python
import json

def safe_extract(json_response: str, field_path: list) -> any:
    data = json.loads(json_response)

    current_dict = data

    try:
        for key in field_path:
            current_dict = current_dict[key]
    except (KeyError, TypeError):
        return None

    return current_dict
```

**Example input / output:**

```text
Input:
- JSON Response: `{"user": {"address": {"city": "New York"}}}`
- Field Path: `["user", "address", "city"]`

Output: `"New York"`

Input:
- JSON Response: `{"user": {"address": {"city": "New York"}}}`
- Field Path: `["user", "state", "city"]`

Output: `None`
```

**Data:**

```json
{
  "user": {
    "address": {
      "city": "New York"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Load JSON into dict
5. Navigate through keys usingfor loop
6. Is key in dictionary?
7. Return value
8. Catch exception
9. Return None
10. End

