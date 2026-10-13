---
type: "practice"
difficulty: "Deep"
---

# Design a form-builder where fields, validators, and conditional visibility ('show state only if country=India') are all data-driven; how do you model conditions without hard-coding field relationships?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
In designing a form builder with low-level design (LLD) principles, we need to ensure that fields, validators, and conditional visibility are data-driven and not hardcoded. This approach promotes flexibility and maintainability. The core idea is to model conditions using a dynamic rules engine that can interpret the relationships between fields based on context such as user input or predefined rules.

#### Triaging the Question
The interviewer is probing your ability to design an extensible and flexible form builder where field definitions, validators, and conditional visibility are driven by data. The key assumptions include:

Fields and their properties (e.g., type, label, placeholder) should be dynamic.
Validators need to be configurable based on the field type or other conditions.
Conditional visibility rules like "show state only if country=India" must be flexible, not hardcoded.

Step-by-Step Solution:

Define the Data Model for Fields: Each form field can have a structure that includes:

id: Unique identifier
type: Type of input (text, checkbox, dropdown, etc.)
label: Text label displayed to the user
placeholder: Placeholder text in the input box
validators: List of validation functions or rules
showConditions: Conditional visibility rules

Dynamic Validation: Validators can be functions that take the field value and return a boolean indicating if it passes validation. Alternatively, they could be more complex objects with error messages.

Conditional Visibility Rules: These should be defined as key-value pairs where keys are conditions (e.g., country), and values are rules for showing or hiding fields based on those conditions.

Implementing the Dynamic Rules Engine:

Use a JSON schema to define the structure of forms.
Parse this schema dynamically during form rendering.
Evaluate visibility rules using an expression evaluator that supports context variables like country.

#### Example
Implementation: Here’s how you might model a field in your data-driven approach:

JSON
{
  "id": "email",
  "type": "text",
  "label": "Email Address",
  "placeholder": "Enter email address",
  "validators": [
    (value) => {
      if (!/\S+@\S+\.\S+/.test(value)) {
        return false;
      }
      return true;
    }
  ],
  "showConditions": {
    "country": ["India"]
  }
}

Rendering the Form:

During rendering, evaluate each field's showConditions against current context (e.g., user session data).
Only display fields that meet their visibility criteria.

#### Closing Summary
and Common Mistakes/Follow-Ups: By keeping the form structure dynamic through JSON schemas and evaluating conditions using an expression engine, we ensure flexibility and maintainability. Key mistakes include hardcoding field relationships or not handling complex logic like nested conditions properly. For follow-ups, you might be asked about performance optimizations for large forms, security considerations (like sanitizing user input), or integration with existing systems.

**Code:**

```json
{
  "fields": [
    {
      "id": "email",
      "type": "text",
      "label": "Email Address",
      "placeholder": "Enter email address",
      "validators": [
        (value) => {
          if (!/\S+@\S+\.\S+/.test(value)) {
            return false;
          }
          return true;
        }
      ],
      "showConditions": {
        "country": ["India"]
      }
    },
    {
      "id": "password",
      "type": "password",
      "label": "Password",
      "placeholder": "Enter password",
      "validators": [
        (value) => value.length >= 8,
        (value, context) => !context.email.includes(value)
      ],
      "showConditions": {
        "country": ["India"]
      }
    }
  ]
}
```

**Example input / output:**

```text
{
  "formDefinition": [
    {
      "id": "email",
      "type": "text",
      "label": "Email Address",
      "placeholder": "Enter email address",
      "validators": [
        (value) => {
          if (!/\S+@\S+\.\S+/.test(value)) {
            return false;
          }
          return true;
        }
      ],
      "showConditions": {
        "country": ["India"]
      }
    },
    {
      "id": "password",
      "type": "password",
      "label": "Password",
      "placeholder": "Enter password",
      "validators": [
        (value) => value.length >= 8,
        (value, context) => !context.email.includes(value)
      ],
      "showConditions": {
        "country": ["India"]
      }
    }
  ]
}
```

**Flow:**

1. Load Form Definition
2. Parse JSON Schema
3. Evaluate Show Conditions
4. Render Visible Fields
5. User Input
6. Validate & Submit

