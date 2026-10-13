---
type: "practice"
difficulty: "Deep"
---

# How would you evolve a public REST field from a string to a structured object in a Python API without breaking existing clients, and what's your deprecation timeline strategy?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pydantic Validation](<../../../../TOOLS/Python/Python_Language/Pydantic_Validation.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve a public REST field from a string to a structured object in a Python API without breaking existing clients, we need to ensure backward compatibility while planning for future changes. This involves carefully managing the transition and setting up a deprecation timeline.

TL;DR:

Current State: The current API accepts strings as input.
Future State: Transitioning this field into a structured object.
Strategy:
Introduce validation and conversion logic during parsing to handle both string and structured object inputs.
Document the change in deprecation notices.
Gradually phase out old code paths over a set timeline.

First, let's triage the question: The core of this problem is about maintaining API stability while implementing new functionality. We need to ensure that existing clients can continue to function without issues, and we should also plan for a gradual migration process. We’ll assume the current version of the API is Version 1 (V1) with string inputs, and we aim to transition it to structured objects by V2.

Step-by-Step Solution
1. Current Implementation
The existing code expects strings as input.
PYTHON
def process_data(data_str):
    # Process the data string in some way
    pass
2. New Requirement
We want to support both strings and structured objects (e.g., dictionaries) as inputs.
3. Backward Compatibility
To ensure backward compatibility, we can add a layer of abstraction that converts the input based on its type.
PYTHON
def process_data(data):
    if isinstance(data, str):  # For existing clients sending strings
        data = convert_string_to_structured_object(data)
    # Proceed with processing the structured object
    pass

def convert_string_to_structured_object(s):
    # Logic to parse and transform string into a structured object (e.g., dictionary)
    return {'key': s}
4. Deprecation Timeline Strategy
Define a clear deprecation timeline.
Notify existing clients about the upcoming changes via documentation updates, version notes, and possibly email notifications.
Plan for gradual phasing out of support for string inputs.

#### Example
Deprecation Notice
MARKDOWN
## Deprecation Notice

We are planning to enhance our API by evolving field `data` from a simple string to a structured object. This change is part of the upcoming V2 release, scheduled for [specific date].

- **Current Version (V1)**: Accepts strings.
- **Future Version (V2)**: Will accept both strings and structured objects.

### Recommendations
- If you are planning to use the `data` field in future integrations, consider using a structured object format immediately.
- For existing clients, we will continue to support string inputs for now but recommend transitioning to structured objects as soon as possible to avoid potential issues down the line.

We appreciate your attention and cooperation during this transition.

#### Closing Summary

In summary, evolving an API field from a string to a structured object requires careful planning and implementation of backward compatibility measures. By introducing type-based validation and conversion logic, we can ensure that existing clients continue to function smoothly while preparing for future changes. A well-defined deprecation timeline helps manage the transition effectively.

#### Common Mistakes/Follow-Ups
Overlooking Existing Clients: Failing to account for current users might lead to compatibility issues.
Inadequate Documentation: Poorly documented deprecations can cause confusion among developers.
Insufficient Testing: Not thoroughly testing the new implementation with both old and new data types could result in bugs.

