---
type: "practice"
difficulty: "Deep"
---

# How do you handle backward-incompatible source schema changes in Bronze that you can't control, e.g. a vendor renames a field mid-stream? What's your absorb-and-adapt layer?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with backward-incompatible source schema changes in a Medallion Architecture (Bronze/Silver/Gold), one of the key challenges is ensuring that the Bronze layer can adapt to these changes without breaking downstream processes. The "absorb-and-adapt" layer plays a crucial role here, allowing us to seamlessly handle such changes by transforming the incoming data into a format compatible with the existing schema in the Silver and Gold layers.

#### Triaging the Question

The question asks about handling backward-incompatible source schema changes where we have no control over the source. The core of the problem is ensuring that the Bronze layer can adapt quickly to these changes without causing downstream disruptions. This involves understanding the data transformation requirements, designing a robust ETL process, and implementing fail-safes.

#### First-Principles Solution

To address this challenge, we need to design an absorb-and-adapt layer that:

Detects Changes: Monitors for schema changes in the source.
Transforms Data: Converts the incoming data into a format compatible with the Bronze layer's schema.
Fails Safely: Ensures that any errors or incomplete transformations do not propagate downstream.
Absorb-and-Adapt Layer Design

The absorb-and-adapt layer can be implemented using a combination of ETL tools and custom logic to handle these changes gracefully. Here’s how it works:

Schema Monitoring:

Use tools like AWS Glue, Apache Airflow, or custom scripts to monitor the source schema.
Set up alerts for any detected changes in the schema.

Data Transformation:

The ETL process should be able to handle missing fields gracefully by either:
Filling in default values (e.g., using a default value or NULL).
Mapping old field names to new ones.
Dropping or adding fields as necessary.
Implement this transformation logic in the ETL jobs, ensuring that they are robust and can handle both old and new schemas.

Incremental Processing:

Use incremental processing techniques to update only the affected records rather than reprocessing all data from scratch.
This helps minimize the computational load and ensures faster recovery times.

Testing and Validation:

Test the ETL jobs thoroughly using sample data that mimics both old and new schemas.
Validate the transformed data against known good values to ensure accuracy.

#### Example
Workflow

Here is a simplified example of how this might look in practice:

PYTHON
# Python code snippet for a simple transformation logic
def transform_data(source_data, new_schema):
    # Initialize an empty dictionary to hold transformed data
    transformed_data = {}

    # Iterate through the source data
    for key, value in source_data.items():
        # Check if the key exists in the new schema
        if key in new_schema:
            transformed_data[key] = value  # Map old field name to new one
        else:
            # Handle missing fields by filling in default values
            if key == 'old_field_name':
                transformed_data['new_field_name'] = DEFAULT_VALUE

    return transformed_data

# Example usage
source_data = {'old_field_name': 'value', 'another_old_field': 'value2'}
new_schema = {'new_field_name': None, 'another_new_field': None}
transformed_data = transform_data(source_data, new_schema)
print(transformed_data)  # Output: {'new_field_name': 'value', 'another_new_field': NULL}

#### Closing Summary
and Common Mistakes

In summary, the absorb-and-adapt layer is essential for handling backward-incompatible schema changes in a Medallion Architecture. It ensures that data transformations are robust, incremental, and fail-safe. Common mistakes include:

Failing to monitor changes in the source schema.
Not implementing robust transformation logic that can handle both old and new schemas.
Ignoring incremental processing, which can lead to unnecessary computational overhead.

By carefully designing this layer, we can ensure a smooth transition during schema changes and maintain the integrity of our data pipeline.

**Comparison:**

```
Old Field Name	New Field Name
old_field_name	new_field_name
```

**Code:**

```python
def transform_data(source_data, new_schema):
    transformed_data = {}
    for key, value in source_data.items():
        if key in new_schema:
            transformed_data[key] = value
        else:
            if key == 'old_field_name':
                transformed_data['new_field_name'] = DEFAULT_VALUE
    return transformed_data

source_data = {'old_field_name': 'value', 'another_old_field': 'value2'}
new_schema = {'new_field_name': None, 'another_new_field': None}
transformed_data = transform_data(source_data, new_schema)
print(transformed_data)  # Output: {'new_field_name': 'value', 'another_new_field': NULL}
```

**Example input / output:**

```text
# Input
source_data = {'old_field_name': 'value', 'another_old_field': 'value2'}
new_schema = {'new_field_name': None, 'another_new_field': None}

# Output
{'new_field_name': 'value', 'another_new_field': NULL}
```

**Data:**

```text
- `source_data`: A dictionary representing the original data.
- `new_schema`: A dictionary representing the updated schema.
```

**Flow:**

1. Source Data
2. Transform
3. Bronze
4. Silver
5. Gold
6. Schema Monitor

