---
type: "practice"
difficulty: "Deep"
---

# How would you evolve units of a numeric field (e.g. cents to dollars, ms to seconds) safely given the type doesn't change?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address evolving units within a numeric field without changing its data type, we need to ensure that transformations are applied consistently and accurately across our system. This involves understanding the current schema, planning the transformation process, implementing the changes, testing thoroughly, and finally ensuring that all systems using this field are updated.

#### Triage
What's Being Probed

The question is asking for a methodical approach to converting units of measurement in an existing numeric field (e.g., cents to dollars or milliseconds to seconds) while maintaining data integrity. The interviewer will be probing your ability to handle schema evolution, versioning, and ensuring backward compatibility.

#### Assumptions
We have a schema that includes the relevant numeric field.
There are no other dependencies on the field's value (e.g., calculations based on this unit).
The transformation is required across multiple systems or versions of our application.
Backward compatibility must be maintained to ensure existing data is not corrupted.
Step-by-Step First Principles Solution
Understand Current Schema: Identify the schema where the numeric field resides, including its current units and any constraints (e.g., minimum/maximum values).
Define New Units: Determine the new unit of measurement you want to use (e.g., dollars instead of cents) and calculate the conversion factor.
Create a Versioned Schema: Introduce a version in your schema registry to represent both old and new units, allowing gradual migration.
Implement Transformation Logic: Write code to apply the transformation logic across all relevant systems or data pipelines.
Test Thoroughly: Perform unit tests, integration tests, and end-to-end tests to ensure correctness.
Migrate Data Gradually: Apply the transformation in a controlled manner, possibly on a subset of the data first.
Monitor and Validate: After migration, monitor system behavior and validate that the new units are used correctly.
Full Solution

Understand Current Schema:

Suppose we have a transactions table with a column amount representing cents.
The schema might look like this:
SQL
CREATE TABLE transactions (
    id INT PRIMARY KEY,
    amount INT, -- in cents
    timestamp TIMESTAMP
);

Define New Units:

We decide to convert the unit from cents to dollars. The conversion factor is 0.01 (since there are 100 cents in a dollar).
New schema version might look like this, with both old and new units:
SQL
CREATE TABLE transactions_v2 (
    id INT PRIMARY KEY,
    amount_cents INT, -- old unit
    amount_dollars DECIMAL(10, 2), -- new unit
    timestamp TIMESTAMP
);

Create a Versioned Schema:

Introduce versioning in the schema registry to ensure compatibility with both versions.

#### Example
versioning entry in a schema registry:
JSON
{
  "version": "1",
  "fields": [
    {"name": "id", "type": "INT"},
    {"name": "amount_cents", "type": "INT"},
    {"name": "timestamp", "type": "TIMESTAMP"}
  ]
}

Implement Transformation Logic:

Write a script to transform existing data from amount_cents to amount_dollars.
PYTHON
import pandas as pd

def convert_amounts(df):
    df['amount_dollars'] = df['amount_cents'] * 0.01
    return df

# Example input dataframe
transactions_df = pd.DataFrame({
    'id': [1, 2, 3],
    'amount_cents': [100, 500, 900]
})

transformed_transactions_df = convert_amounts(transactions_df)
print(transformed_transactions_df)

Test Thoroughly:

Perform unit tests on the transformation logic.
Validate that all calculations and constraints are respected in both new and old units.

Migrate Data Gradually:

Start by converting a small subset of data to ensure no issues arise.
SQL
INSERT INTO transactions_v2 (id, amount_cents, timestamp)
SELECT id, amount_cents, timestamp FROM transactions LIMIT 10;

Monitor and Validate:

Monitor system performance post-migration.
Validate that the new units are used correctly in all downstream systems.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, evolving units of a numeric field requires careful planning to ensure data integrity and backward compatibility. Key steps include understanding current schema, defining new units, creating versioned schemas, implementing transformation logic, testing thoroughly, migrating data gradually, and validating the changes. Common mistakes include failing to test fully or migrating too aggressively without proper validation.

For follow-ups, you might be asked about handling concurrent access during migration, dealing with complex business rules that depend on the unit, or strategies for rolling back if issues arise.

**Comparison:**

| id | amount_cents | amount_dollars |
|----|--------------|----------------|
| 1  | 100          | 1.0            |
| 2  | 500          | 5.0            |
| 3  | 900          | 9.0            |

**Code:**

```python
import pandas as pd

def convert_amounts(df):
    df['amount_dollars'] = df['amount_cents'] * 0.01
    return df

# Example input dataframe
transactions_df = pd.DataFrame({
    'id': [1, 2, 3],
    'amount_cents': [100, 500, 900]
})

transformed_transactions_df = convert_amounts(transactions_df)
print(transformed_transactions_df)
```

**Example input / output:**

```

Input:

PYTHON
Copy
import pandas as pd

def convert_amounts(df):
    df['amount_dollars'] = df['amount_cents'] * 0.01
    return df

# Example input dataframe
transactions_df = pd.DataFrame({
    'id': [1, 2, 3],
    'amount_cents': [100, 500, 900]
})

transformed_transactions_df = convert_amounts(transactions_df)
print(transformed_transactions_df)

Output:

PLAINTEXT
Copy
   id  amount_cents  amount_dollars
0   1           100             1.0
1   2           500             5.0
2   3           900            9.0
```

**Data:**

```sql
CREATE TABLE transactions (
    id INT PRIMARY KEY,
    amount INT, -- in cents
    timestamp TIMESTAMP
);

CREATE TABLE transactions_v2 (
    id INT PRIMARY KEY,
    amount_cents INT, -- old unit
    amount_dollars DECIMAL(10, 2), -- new unit
    timestamp TIMESTAMP
);
```

**Flow:**

1. Yes
2. No
3. Yes
4. Understand Current Schema
5. Create New Units?
6. Define Versioned Schema
7. Implement TransformationLogic
8. Test Thoroughly
9. Migrate Data Gradually
10. Monitor and Validate
11. Deploy and Go Live
12. Handle Concurrent Access?
13. J
14. Implement ConcurrencyControl

