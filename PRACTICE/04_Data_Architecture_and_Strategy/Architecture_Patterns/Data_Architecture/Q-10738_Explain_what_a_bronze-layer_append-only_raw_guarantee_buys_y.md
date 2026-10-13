---
type: "practice"
difficulty: "Intermediate"
---

# Explain what a bronze-layer 'append-only raw' guarantee buys you when you later discover a bug in your silver transformation.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Immutable Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/GDPR_Deletion_in_Immutable_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data architecture, especially within the context of a data lake or warehouse setup, a "bronze layer" is often used to store untransformed raw data with an append-only policy. This means that once data is ingested into this layer, it can only be appended to, not modified or deleted. When you later discover a bug in your silver transformation layer (which processes the bronze-layer data), the bronze layer provides several key benefits.

Firstly, the "append-only raw" guarantee ensures that the original state of the data remains intact and unaltered. This is crucial because it allows you to investigate and correct any issues without affecting historical data transformations. You can keep a record of how your dataset looked when the bug was introduced, which is invaluable for debugging.

Secondly, having a versioned raw data store helps maintain consistency across different transformations. If there's an issue in one transformation pipeline, it’s easier to pinpoint exactly where and why problems arose by referring back to the bronze layer. This can save significant time and effort compared to starting from scratch or dealing with altered datasets.

Lastly, the append-only nature of the bronze layer provides a fail-safe mechanism for data quality assurance. It ensures that no accidental deletions or modifications occur, which is particularly important in environments where data integrity is critical, such as financial reporting systems or regulatory compliance scenarios.

In summary, the bronze layer acts as a robust foundation upon which you can build and refine your transformations, providing flexibility to correct errors without compromising the integrity of original data. It enables easier debugging, ensures consistency across transformations, and protects against accidental alterations.

**Comparison:**

|Bronze Layer Table Schema|
|---|
|timestamp (datetime)|
|transaction_id (integer)|
|amount (float)|

**Code:**

```python
# Example pseudo-code for ingesting data into bronze layer
def ingest_data_into_bronze_layer(data):
    # Code to ensure append-only property
    if not is_append_only:
        raise Exception("Bronze layer should be append-only.")
    # Log the ingestion process
    logging.info(f"Ingested raw data: {data}")
```

**Example input / output:**

```text
Input:
Timestamp: 2023-10-01T14:30:00Z, Transaction ID: 123456789, Amount: 150.25

Output:
Successfully ingested raw data: {"timestamp": "2023-10-01T14:30:00Z", "transaction_id": 123456789, "amount": 150.25}
```

**Data:**

```yaml
# Example schema for bronze-layer dataset
raw_data:
  - timestamp: "2023-10-01T14:30:00Z"
    transaction_id: 123456789
    amount: 150.25
```

**Flow:**

1. Yes
2. No
3. Raw Data
4. Is it Append-Only?
5. Ingest Raw Data
6. Error, Append-Only PolicyViolated
7. Bronze Layer Storage

