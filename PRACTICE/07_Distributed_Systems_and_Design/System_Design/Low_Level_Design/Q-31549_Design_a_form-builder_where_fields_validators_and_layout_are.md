---
type: "practice"
difficulty: "Intermediate"
---

# Design a form-builder where fields validators and layout are configurable. What is the object model?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Extensible Rule Engines](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Extensible_Rule_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a form-builder with configurable fields, validators, and layouts, we need to ensure that our solution is flexible enough to accommodate various configurations while maintaining simplicity and performance. The object model should encapsulate the essence of how forms are built, validated, and rendered.

#### Triaging the Question

The core question here is about designing an object-oriented structure for a form-builder. We need to define classes or objects that represent different components such as fields, validators, and layouts. Additionally, we should consider how these components interact with each other to build, validate, and display forms dynamically.

#### Assumptions
and Key Concepts
Dynamic Forms: The form-builder needs to support dynamic generation of forms based on configuration.
Configurability: Fields, validators, and layout rules can be changed at runtime without modifying the core codebase.
Reusability: Components should be reusable across different parts of the application.

#### First-Principles Solution

We will start by defining three main classes: Field, Validator, and Layout. Each field can have its own set of validators, and layouts define how fields are arranged on a form. The interaction between these components will involve setting up forms with fields, applying validators to the fields, and rendering the layout.

1. Field Class

Fields represent individual data entry points in the form.

PYTHON
class Field:
    def __init__(self, name, field_type):
        self.name = name
        self.field_type = field_type
        self.validators = []

    def add_validator(self, validator):
        self.validators.append(validator)

    def validate(self, value):
        for validator in self.validators:
            if not validator(value):
                return False
        return True
2. Validator Class

Validators are functions that check the validity of input values.

PYTHON
class Validator:
    @staticmethod
    def is_required(value):
        return bool(value.strip())

    @staticmethod
    def is_email(value):
        import re
        return re.match(r"[^@]+@[^@]+\.[^@]+", value) is not None

    # Add more validators as needed
3. Layout Class

Layouts define the arrangement of fields on a form.

PYTHON
class Layout:
    def __init__(self, fields):
        self.fields = fields

    def display(self):
        for field in self.fields:
            print(f"{field.name}: {getattr(field, field.field_type)}")

    # More methods to customize layout rendering as needed
4. Form Class

Forms aggregate fields and layouts.

PYTHON
class Form:
    def __init__(self, name, fields):
        self.name = name
        self.fields = fields

    def add_field(self, field):
        self.fields.append(field)

    def validate_form(self, data):
        valid = True
        for field in self.fields:
            if not field.validate(getattr(data, field.name)):
                print(f"Validation failed for {field.name}")
                valid = False
        return valid

    def display_layout(self):
        layout = Layout([field for field in self.fields])
        layout.display()

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the object model we designed includes Field, Validator, and Layout classes. These components interact to build a dynamic form that can be validated and displayed based on configurable rules. Common mistakes might include not properly handling edge cases for validators or not allowing flexible addition of new field types in the future.

For follow-ups, you could explore:

Performance Optimization: How would you handle large forms with thousands of fields?
Security: What measures should be taken to prevent injection attacks or other security vulnerabilities?

