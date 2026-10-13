---
type: "practice"
difficulty: "Expert"
---

# How would you design the lake's contract with AIOps ML so a schema or unit change is caught at the data-contract gate rather than surfacing as silent model drift weeks later?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Curated Observability Datasets](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Curated_Observability_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In designing an Observability Data Lake to ensure that changes in schemas or units are caught early, we need to focus on establishing robust data contracts and monitoring mechanisms. This ensures that any deviations from expected data structures are detected promptly, preventing silent model drift. The key is to integrate the data lake’s schema validation with machine learning (ML) models through a well-defined pipeline.

The primary goal here is to catch contract violations at the data-contract gate rather than allowing them to propagate silently and eventually cause issues in ML models. We'll start by defining what constitutes a "contract" and then outline how we can implement this design using first-principles reasoning.

**Comparison:**

```
Schema Validity	Incoming Data
True	{"metric1": 42, "metric2": "value"}
False	{"metric1": 42}
```

**Code:**

```python
# Example of a simple schema validation function
def validate_schema(data, expected_schema):
    return all(k in data for k in expected_schema)
```

**Example input / output:**

```text
- **Input**:
  - `expected_schema`: {"type": "object", "properties": {"metric1": {"type": "number"}, "metric2": {"type": "string"}}, "required": ["metric1", "metric2"]}
  - `incoming_data`: {"metric1": 42, "metric2": "value"}
- **Output**:
  - Validation Result: True
- **Input**:
  - `expected_schema`: {"type": "object", "properties": {"metric1": {"type": "number"}, "metric2": {"type": "string"}}, "required": ["metric1"]}
  - `incoming_data`: {"metric1": 42, "metric2": "value"}
- **Output**:
  - Validation Result: False
```

**Data:**

```
expected_schema (JSON Schema):
JSON
Copy
{
  "type": "object",
  "properties": {
    "metric1": {"type": "number"},
    "metric2": {"type": "string"}
  },
  "required": ["metric1", "metric2"]
}
incoming_data (Sample JSON):
JSON
Copy
{
  "metric1": 42,
  "metric2": "value"
}
```

**Flow:**

1. Data Ingestion
2. Schema Validation
3. Alert if Violation Detected
4. Log Issue and Notify Stewards

