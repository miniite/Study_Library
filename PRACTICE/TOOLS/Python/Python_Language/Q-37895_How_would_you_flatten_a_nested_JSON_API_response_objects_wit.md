---
type: "practice"
difficulty: "Intermediate"
---

# How would you flatten a nested JSON API response (objects within objects, arrays of objects) into flat tabular rows in Python for loading into a warehouse?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parsing JSON Safely](<../../../../TOOLS/Python/Python_Language/Parsing_JSON_Safely.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To flatten a nested JSON API response and convert it into a flat tabular format suitable for a data warehouse using Python, we need to break down the structure recursively and ensure all nested objects are transformed into flat rows with appropriate column names. This involves understanding the hierarchy of the JSON data, mapping keys appropriately, and managing duplicate or derived key scenarios.

#### Triaging the Question

The core question is about transforming complex nested JSON structures into a tabular format for easier analysis and storage in a data warehouse. The interviewer might be probing your understanding of Python's data manipulation capabilities, recursive programming concepts, and how to handle edge cases like duplicated keys.

#### Assumptions
The input JSON structure can vary widely.
Keys within objects may overlap or be unique.
The output should maintain the relationships between different entities (e.g., parent-child associations).
Step-by-Step Solution

We will start by defining a recursive function to traverse through the nested JSON structure and flatten it. We'll use Python's built-in data structures like dictionaries, lists, and sets for this purpose.

1. Define the Recursive Function

The key idea is to break down each object in the JSON into its attributes and values while managing potential conflicts using suffixes or other methods.

PYTHON
import json

def flatten_json(data, parent_key='', sep='_'):
    items = []
    if isinstance(data, dict):
        for k, v in data.items():
            new_key = f"{parent_key}{sep}{k}" if parent_key else k
            items.extend(flatten_json(v, new_key, sep=sep))
    elif isinstance(data, list):
        i = 0
        for item in data:
            i += 1
            items.extend(flatten_json(item, f'{parent_key}_{i}', sep=sep))
    else:
        items.append((parent_key, data))
    return items

def flatten_dict(data):
    flattened_items = flatten_json(data)
    # Convert list of tuples to a dictionary for easier handling
    flattened_data = {k: v for k, v in flattened_items}
    return flattened_data

# Example usage
nested_json = {
    "user": {
        "id": 1,
        "name": "John Doe",
        "address": {
            "street": "123 Main St",
            "city": "Anytown"
        },
        "phone_numbers": [
            {"number": "555-1234"},
            {"number": "555-5678"}
        ]
    }
}

flat_data = flatten_dict(nested_json)
print(json.dumps(flat_data, indent=2))
2. Handle Duplicates and Conflicts

If there are duplicate keys, we can manage them by appending an index or using a more sophisticated approach like hashing.

#### Closing Summary
+ Common Mistakes/Follow-Ups

This solution effectively flattens nested JSON data into a tabular format suitable for data warehouses. Key points include:

Recursively traversing the JSON structure.
Handling lists and dictionaries appropriately.
Managing duplicate keys to avoid overwriting values.

#### Common mistakes
include:

Not handling lists properly, leading to incorrect nesting.
Ignoring potential conflicts or duplicates in key names.
Overlooking edge cases like empty objects or arrays.

#### Follow-ups
could include discussions on performance optimizations, dealing with large datasets, and integrating this transformation into ETL pipelines.

