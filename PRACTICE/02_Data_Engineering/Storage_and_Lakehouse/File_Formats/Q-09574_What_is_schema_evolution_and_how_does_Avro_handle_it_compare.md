---
type: "practice"
difficulty: "Deep"
---

# What is schema evolution and how does Avro handle it compared to Parquet?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Avro Schema Evolution](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Schema evolution refers to the process of modifying a data schema over time without disrupting existing data. It's crucial in environments where data models are not static but evolve as new requirements emerge or old ones change.

In this context, let’s compare Avro and Parquet’s approaches to handling schema evolution:

Avro

Avro is a data serialization system that supports full schema evolution capabilities. This means you can add, remove, or modify fields in the schema without affecting existing data, which resides in files with versions of the schema metadata.

Schema Versioning: Avro allows maintaining multiple versions of schemas and uses logical record keys to track changes.
Compatibility Modes:
Backward Compatible: You can add new fields but must not remove or change existing ones.
Forward Compatible: New fields are added, allowing future readers to ignore them.
Strict Mode: Ensures that the schema is strictly adhered to, which might fail on mismatches.
Parquet

Parquet, on the other hand, has a more limited approach to schema evolution:

Schema Versioning: While it supports schema versioning through the use of metadata, changes are not as flexible.
Compatibility Modes:
Parquet’s compatibility mode is less strict and does not enforce as many constraints compared to Avro.
Step-by-Step Reasoning
Summary

Avro excels in schema evolution due to its robust version control and flexibility, while Parquet provides a more structured approach but with limitations.

#### Triaging the Question

The question asks how Avro handles schema evolution compared to Parquet. The core of this query is understanding both systems’ mechanisms for managing changes over time.

#### Assumptions
Both formats support versioning.
We are comparing their handling of schema changes in terms of backward, forward, and strict compatibility modes.

#### First Principles Solution

Avro’s Approach:

Schema Version Control: Each record is tagged with a schema ID, allowing for flexible updates without breaking existing data.
Backward Compatibility: Allows adding new fields or modifying schemas.
Forward Compatibility: Adds new fields while old readers can ignore them.
Strict Mode: Enforces strict adherence to the latest schema.

Parquet’s Approach:

Schema Versioning: Maintains backward compatibility by using metadata to track changes but with more rigid constraints.
Backward Compatibility: Allows adding new fields or data types that can be ignored by older readers.
Forward Compatibility: Limited; schema changes might require manual intervention.

#### Closing Summary
and Common Mistakes

Key Takeaways:

Avro offers a more flexible approach to schema evolution, making it ideal for dynamic environments where frequent changes are expected.
Parquet is better suited for applications that benefit from stricter schema adherence but also need occasional updates.

#### Common Mistakes

Overlooking the importance of schema compatibility when transitioning between systems.
Assuming strict modes in both formats provide the same level of flexibility.

#### Example

Let’s consider a practical example where you have an initial schema with fields id, name, and age. You later decide to add a new field address:

Avro:

Versioned schema: {"id": "int", "name": "string", "age": "int", "address": "string"}
Existing data remains intact, with readers handling the absence of the address field gracefully.

Parquet:

Schema changes must be handled carefully to avoid breaking existing data. Adding a new column might require manual updates or intermediate transformations.

#### Conclusion

Understanding schema evolution is crucial for choosing the right file format based on your project's needs. Avro’s robust versioning and flexible compatibility modes make it an excellent choice for environments where frequent schema changes are inevitable, while Parquet provides a more structured approach better suited for applications that require stricter adherence to schemas.

**Comparison:**

| Schema Type | Backward Compatibility | Forward Compatibility | Strict Mode |
|-------------|-----------------------|----------------------|------------|
| Avro        | Supported             | Enabled              | Enforced   |
| Parquet     | Limited               | Basic                | Not Enforced |

**Code:**

```
Example of reading avro data with schema evolution

from pyarrow import parquet as pq

def read_avro_with_evolution(file_path): table = pq.read_table(file_path, preserve_schema_on_write=False) print(table.to_pandas())

CODE
Copy

```python
# Example Parquet handling with compatibility considerations
def handle_parquet_compatibility(file_path):
    schema = {"fields": [{"name": "id", "type": "int"}, {"name": "name", "type": "string"}]}
    table = pq.read_table(file_path, schema=schema)
    print(table.to_pandas())
```

**Example input / output:**

```
Example input->output for Avro with schema evolution

Input (initial data): {"id": 1, "name": "John Doe", "age": 30}

Output (with updated schema): {"id": 1, "name": "John Doe", "age": 30, "address": null}

CODE
Copy

```plaintext
# Example input->output for Parquet with compatibility considerations
Input (initial data):
{"id": 1, "name": "John Doe"}

Output (with updated schema, handling missing address):
{"id": 1, "name": "John Doe", "address": null}
```

**Data:**

```json
{
  "initial_schema": {
    "fields": [
      { "name": "id", "type": "int" },
      { "name": "name", "type": "string" },
      { "name": "age", "type": "int" }
    ]
  },
  "updated_schema": {
    "fields": [
      { "name": "id", "type": "int" },
      { "name": "name", "type": "string" },
      { "name": "age", "type": "int" },
      { "name": "address", "type": "string" }
    ]
  }
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. No
6. Read Parquet File
7. Is Schema Evolving?
8. Handling Strict Mode?
9. Perform Manual Updates
10. Add New Fields, Ignore OldReaders
11. Is Data Incompatible?
12. Handle Schema MismatchManually
13. Process Data Normally

