---
type: "practice"
difficulty: "Intermediate"
---

# What is a data schema, and how would you enforce that incoming records match an expected schema in Python before writing them to a target?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Schema Evolution in Python](<../../../../TOOLS/Python/Python_Language/Handling_Schema_Evolution_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In simple terms, a data schema defines the structure of data by specifying the fields (columns) along with their types and constraints. When dealing with structured data, especially in Python, ensuring that incoming records conform to this predefined schema is crucial for maintaining data integrity and consistency.

#### Triage

The question asks us to explain what a data schema is and how we can enforce matching an expected schema before writing data to a target storage system (like a database or file). We need to consider how to validate the structure of incoming records in Python, ensuring they adhere to the defined schema. This involves understanding the structure of the data, checking for valid values, and handling potential mismatches.

#### First Principles Solution
Step 1: Define the Schema

First, we define our schema by specifying the expected fields and their types. For example, if we're dealing with a user record, we might have fields like id, name, email, and age.

PYTHON
schema = {
    'id': int,
    'name': str,
    'email': str,
    'age': int
}
Step 2: Validate Incoming Records

Next, we write a function to validate each incoming record against the schema. This involves checking that all required fields are present and have valid types.

PYTHON
def validate_record(record, schema):
    for field_name, expected_type in schema.items():
        if field_name not in record:
            return False
        if not isinstance(record[field_name], expected_type):
            return False
    return True
Step 3: Enforce the Schema Before Writing Data

Finally, we integrate this validation logic into our data processing pipeline to ensure that only records conforming to the schema are written to the target storage.

#### Example
Implementation
PYTHON
# Define a sample record and schema
record = {'id': 1, 'name': 'Alice', 'email': 'alice@example.com', 'age': 25}
schema = {
    'id': int,
    'name': str,
    'email': str,
    'age': int
}

# Validate the record against the schema
if validate_record(record, schema):
    print("Record is valid. Writing to target storage...")
else:
    print("Invalid record. Discarding...")

# Example of a validation failure
invalid_record = {'id': 1, 'name': 25, 'email': 'alice@example.com', 'age': 'twenty-five'}
if validate_record(invalid_record, schema):
    print("Record is valid.")
else:
    print("Invalid record. Discarding...")

#### Closing Summary
and Common Mistakes

By defining a clear schema and validating incoming records, we ensure data consistency and reliability. Common mistakes include not handling all edge cases (e.g., missing required fields) or failing to account for unexpected types of data. Additionally, always testing the validation function with both valid and invalid records is crucial.

Final Check

This solution covers defining a schema, writing a validation function, and integrating it into the data processing pipeline.

