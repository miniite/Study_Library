---
type: "practice"
difficulty: "Deep"
---

# How would you retrofit data quality onto a legacy mainframe-fed warehouse where you can't change the producers and batch windows are fixed and immovable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Quality SLAs and Rule Testing](<../../../../02_Data_Engineering/Data_Quality/Quality_Frameworks_and_SLAs/Quality_SLAs_and_Rule_Testing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To retrofits data quality onto a legacy mainframe-fed warehouse with fixed and immovable batch windows, we need to adopt a layered approach focusing on pre-processing, ingestion validation, and post-ingestion monitoring. This involves implementing a robust data quality framework and Service Level Agreements (SLAs) that align with the constraints of the existing environment.

#### Triaging the Question

The core issue here is integrating modern data quality practices into an older system without altering its producers or batch windows. We need to establish checks and balances within our pipeline to ensure data integrity, while also setting clear SLAs to manage expectations around data availability and accuracy.

#### Assumptions
and Context
Batch Windows: The mainframe generates fixed batches at specific times.
Read-Only Environment: Direct changes to the mainframe's producers are not possible.
Post-Ingestion Monitoring: Real-time checks during batch processing might be limited or impossible due to constraints.

#### First Principles Solution

Pre-Proceed Validation:

Data Transformation Rules: Define rules for transforming raw data into a more usable format before ingestion.
Schema Validation: Ensure incoming data adheres to expected schemas, which can help catch common issues early in the pipeline.

Ingestion Quality Checks:

Batch Ingestion Scripts: Develop scripts that run during batch windows to validate key metrics such as completeness, accuracy, and consistency.
Sampling Mechanisms: Implement sampling techniques to check a subset of data for anomalies without disrupting the full ingestion process.

Post-Ingestion Monitoring:

Data Quality Metrics: Continuously monitor post-ingestion data using KPIs like error rates, latency, and completeness.
Alerting Systems: Set up alert systems that notify stakeholders when data quality thresholds are breached, allowing timely corrective action.

SLA Establishment:

Define SLAs: Clearly define service level agreements for key performance indicators (KPIs) such as data availability, accuracy, and processing time.
Stakeholder Communication: Engage with stakeholders to ensure they understand the agreed-upon service levels and their implications.

Operationalizing the Framework:

Automated Testing: Use automated testing tools to regularly validate data transformations and ingestion processes.
Logging & Auditing: Ensure comprehensive logging and auditing mechanisms are in place to track data quality issues over time.

#### Closing Summary

By implementing a well-structured data quality framework that includes pre-processing validation, ingestion checks, and post-ingestion monitoring, we can mitigate the risks associated with integrating modern data quality practices into an older mainframe-fed warehouse. This approach ensures that while we cannot change the producers or batch windows, we can still maintain high standards of data integrity and reliability.

#### Common Mistakes/Follow-Ups
Over-Reliance on Post-Ingestion Monitoring: Ignoring pre-processing and ingestion checks can lead to undetected issues.
Neglecting SLA Communication: Not involving stakeholders in defining and agreeing upon SLAs can result in misaligned expectations and poor service delivery.
Lack of Automation: Manual processes for validation and monitoring can be error-prone and time-consuming.

**Comparison:**

```
Data Quality Metric	Definition	Target
Error Rate	Proportion of erroneous records in a batch	≤ 5%
Completeness	Percentage of expected fields present	≥ 95%
Latency	Time taken to process and validate data	< 10s
```

**Code:**

```python
# Example Python script for batch ingestion validation
def validate_batch(data):
    # Implement schema validation
    if not validate_schema(data):
        raise ValueError("Schema mismatch detected")

    # Implement sampling mechanism
    sample_data = random.sample(data, min(100, len(data)))
    for row in sample_data:
        if not is_valid_row(row):
            print(f"Invalid row: {row}")

# Example validation functions
def validate_schema(data):
    schema_keys = ["field1", "field2", "field3"]
    return all(key in data for key in schema_keys)

def is_valid_row(row):
    # Implement logic to check if a row meets validity criteria
    return True
```

**Example input / output:**

```text
Input:
- Batch data from mainframe

Output:
- Validated batch data
- Alerts for invalid rows or schema mismatches
- Metrics on data quality (e.g., error rates, completeness)
```

**Data:**

```json
{
  "data_schema": {
    "fields": ["id", "name", "value"],
    "constraints": [
      { "type": "required", "field": "id" },
      { "type": "length", "field": "name", "min": 3, "max": 50 }
    ]
  },
  "batch_data": [
    {"id": 1, "name": "John Doe", "value": 200},
    {"id": 2, "name": "", "value": 500}  // Invalid row
  ]
}
```

**Flow:**

1. Pass
2. Yes
3. Invalid
4. Valid
5. Fail
6. Batch Received
7. Validate Schema
8. Sample Data
9. Check Rows
10. Alert Sent
11. Store in Warehouse
12. Handle Error

