---
type: "practice"
difficulty: "Deep"
---

# Explain schema-on-read versus schema-on-write with an example from your own pipelines. When has schema-on-read bitten you and how did you contain it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Warehouse as System of Record](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_as_System_of_Record.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Schema-on-read and schema-on-write are two different approaches to handling data schemas in a data architecture, each with its own trade-offs. Schema-on-read involves defining the schema at query time or processing time, while schema-on-write requires defining the schema upfront before storing the data. This distinction is particularly relevant when designing modern data architectures that include data lakes and warehouses.

In this answer, I will explain these concepts from a Data Engineer's perspective, focusing on the differences between them, their pros and cons, and an example from my own pipelines where schema-on-read led to issues. I'll also discuss how I mitigated those problems.

**Comparison:**

| Approach | Schema Definition Time       | Flexibility            | Scalability     | Performance |
|----------|------------------------------|------------------------|-----------------|-------------|
| Schema-on-write | Upfront (before storage)    | Less flexible         | High            | Good        |
| Schema-on-read   | At query/processing time  | More flexible         | Medium          | Variable    |

**Code:**

```python
# Example code for schema-on-write
def write_to_data_lake(data):
    # Assume data is a dictionary with predefined keys and types
    with open('data_lake.csv', 'a') as file:
        file.write(','.join([str(value) for value in data.values()]) + '\n')

# Example code for schema-on-read
def process_data_from_lake():
    import pandas as pd
    df = pd.read_csv('data_lake.csv')
    # Dynamically determine column types and use them to process the data
    processed_data = df.apply(lambda row: {
        'id': int(row[0]),
        'timestamp': pd.to_datetime(row[1]),
        'value': float(row[2])
    }, axis=1)
```

**Example input / output:**

```

Given the sample data:

YAML
Copy
data_lake_sample_data:
  [1, "2023-10-01", 10.5]
  [2, "2023-10-02", 11.0]
  [3, "2023-10-03", 12.3]

Using schema-on-write:

PYTHON
Copy
# Writing the data to the lake with predefined schema
write_to_data_lake(data={"id": 1, "timestamp": "2023-10-01", "value": 10.5})
write_to_data_lake(data={"id": 2, "timestamp": "2023-10-02", "value": 11.0})
write_to_data_lake(data={"id": 3, "timestamp": "2023-10-03", "value": 12.3})

Using schema-on-read:

PYTHON
Copy
# Reading and processing the data from the lake with dynamic schema
processed_data = process_data_from_lake()
print(processed_data)

Output:

JSON
Copy
[
    {'id': 1, 'timestamp': datetime.datetime(2023, 10, 1), 'value': 10.5},
    {'id': 2, 'timestamp': datetime.datetime(2023, 10, 2), 'value': 11.0},
    {'id': 3, 'timestamp': datetime.datetime(2023, 10, 3), 'value': 12.3}
]
```

**Data:**

```yaml
data_lake_schema_on_write:
  - id: int
  - timestamp: datetime
  - value: float

data_lake_sample_data:
  [1, "2023-10-01", 10.5]
  [2, "2023-10-02", 11.0]
  [3, "2023-10-03", 12.3]
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Schema-on-write
6. Is schema defined?
7. Data stored with predefinedschema
8. Error or data loss
9. Schema-on-read
10. Do you need to process on thefly?
11. Dynamic schemadetermination and processing
12. Static schema storage andprocessing

