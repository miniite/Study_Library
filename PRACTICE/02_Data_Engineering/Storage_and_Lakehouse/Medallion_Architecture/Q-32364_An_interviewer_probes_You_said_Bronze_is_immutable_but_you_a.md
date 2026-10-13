---
type: "practice"
difficulty: "Deep"
---

# An interviewer probes: 'You said Bronze is immutable — but you also apply dedup and typing. Isn't that already transformation?' How do you defend your layer boundary?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The key to understanding why Bronze in Medallion Architecture can have transformations like deduplication and typing lies in recognizing that these operations are purely data cleaning and validation steps, not altering the source data integrity. They ensure consistency before moving data into more refined layers.

#### Triage
The interviewer is probing whether our understanding of layer boundaries in Medallion Architecture holds up under scrutiny. Specifically, they question if applying deduplication and typing to Bronze, which is meant to be immutable, contradicts its purpose. Let's clarify the roles and constraints of each layer.

Step-by-Step Solution:

Understanding Layer Boundaries: In Medallion Architecture, data flows through three layers: Bronze (raw), Silver (cleaned and validated), and Gold (derived insights). Each layer has a specific role:

Bronze Layer: This is the raw, unprocessed data directly ingested from various sources.
Silver Layer: Data here undergoes cleaning, validation, deduplication, and some basic transformations to ensure consistency before moving into the final layers.
Gold Layer: Here, more complex transformations and aggregations occur, providing business insights.

Immutable Nature of Bronze Layer:

The core concept is that Bronze data should not be altered or processed further in this layer. It serves as a snapshot of raw incoming data, ensuring that we always have the most up-to-date raw state.
However, "immutable" does not mean no changes can ever occur; it means these changes are strictly confined to cleaning and validation steps that do not compromise the integrity of the original source.

Deduplication and Typing in Bronze Layer:

Deduplication ensures that we only store unique records, reducing redundancy.
Typing involves defining data structures explicitly (e.g., ensuring a column is always of string type) to maintain consistency and facilitate processing downstream.
These operations are critical for maintaining the quality of data before it moves into Silver. They do not alter the fundamental nature of the raw data or its source integrity.

Maintaining Layer Boundaries:

By performing deduplication and typing in Bronze, we ensure that when moving to Silver, we have already cleansed the data. This allows us to focus on more complex transformations without worrying about basic issues like duplicates or inconsistent types.
These steps are crucial for ensuring that our data pipelines remain robust and scalable.

#### Conclusion

Deduplication and typing in Bronze do not violate its immutable nature because they serve as a form of data validation and cleaning, not transformation. They prepare the data for further processing without altering the fundamental structure or integrity of the source.
Properly managing these steps ensures that each layer serves its intended purpose: raw data storage (Bronze), clean data handling (Silver), and insight generation (Gold).

#### Closing Summary
Understanding the roles and constraints of each layer in Medallion Architecture is essential. Deduplication and typing are necessary for maintaining data quality and consistency but do not compromise the immutability of Bronze by altering its fundamental nature.

#### Common Mistakes/Follow-Ups

Misunderstanding the term "immutable" as strictly prohibiting any changes, leading to confusion about necessary cleaning steps.
Overlooking the importance of data validation in preparing raw data for further processing.

**Comparison:**

| Layer   | Description                             | Key Operations                         |
|---------|-----------------------------------------|---------------------------------------|
| Bronze  | Raw, unprocessed data                    | Deduplication, Typing                 |
| Silver  | Cleaned and validated data               | Additional transformations             |
| Gold    | Derived insights and business intelligence | Aggregations, derivations              |

**Code:**

```python
# Example code snippet for deduplication and typing in Bronze layer
import pandas as pd

def clean_bronze_data(df):
    # Deduplicate records based on key columns
    df = df.drop_duplicates(subset=['key_column'])

    # Define data types explicitly
    df['column_name'] = df['column_name'].astype(str)

    return df
```

**Example input / output:**

| Input DataFrame | Output DataFrame |
|-----------------|------------------|
| id | name | age | id | name | age |
| 1 | John Doe | 30 | 1 | John Doe | 30 |
| 2 | Jane Smith | 25 | 2 | Jane Smith | 25 |

**Data:**

```json
{
  "bronze_data": [
    {"id": 1, "name": "John Doe", "age": "30"},
    {"id": 2, "name": "Jane Smith", "age": "25"},
    {"id": 1, "name": "John Doe", "age": "30"}  // Duplicate record
  ]
}
```

**Flow:**

1. Raw Data
2. Bronze Layer - Deduplication& Typing
3. Silver Layer - Validation &Transformation
4. Gold Layer - InsightsGeneration

