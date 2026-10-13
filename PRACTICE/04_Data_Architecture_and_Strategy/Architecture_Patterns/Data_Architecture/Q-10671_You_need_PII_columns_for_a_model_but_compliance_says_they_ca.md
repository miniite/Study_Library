---
type: "practice"
difficulty: "Deep"
---

# You need PII columns for a model but compliance says they can't sit unmasked in the lake. How do you get usable features while respecting masking/tokenization?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Proving PII Residency](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Proving_PII_Residency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of storing Personally Identifiable Information (PII) columns masked or tokenized to comply with data protection regulations, we need a robust data architecture that ensures both usability and compliance. The core idea is to leverage a combination of a Data Lake for raw unprocessed data and Data Warehouses for processed, aggregated, and anonymized data. This approach allows us to maintain the integrity of our machine learning (ML) models while adhering to strict privacy policies.

The question probes your understanding of data architecture principles, specifically how to handle sensitive information in a way that supports ML model training without compromising regulatory requirements. Key assumptions include: 1) we have PII columns needed for the ML model but must mask them before they enter our systems; and 2) compliance regulations require such masking.

Let's break this down from first principles:

Step-by-Step Solution
1. Data Lake (Raw Storage)
Store raw, unprocessed data in a Data Lake.
Ensure that PII columns are masked or tokenized before they enter the Data Lake to comply with regulations.
Use tools like Apache Beam for data ingestion pipelines where sensitive information is processed on-the-fly.
2. Data Processing Pipeline
Implement a pipeline using Apache Spark, Flink, or another stream processing engine.
Mask or tokenize PII columns in real-time as the data flows into the Data Lake.
Store masked data directly within the Data Lake for future use by ML models.
3. Data Warehouse (Processed Storage)
Extract and transform the masked data from the Data Lake to a Data Warehouse using tools like Apache Airflow or Talend.
Perform aggregation, anonymization, and further transformations in the Data Warehouse to ensure that PII is fully masked and unusable for re-identification.
Use these transformed datasets for training ML models.
4. Feature Engineering
Develop a feature extraction process that works on the masked data stored in the Data Warehouse.
Ensure that all features generated from the masked data are usable by your ML model while respecting compliance requirements.
Detailed Solution
Data Lake Architecture
Raw Data Ingestion
Data Lake
Real-Time Masking
Transformed & Masked Data
Data Warehouse Architecture
Extract & Transform
Data Warehouse
Aggregation & Anonymization
Usable Features for ML Models

#### Closing Summary
and Common Mistakes

By combining a Data Lake with robust data processing pipelines and a Data Warehouse, we can ensure that PII columns are masked or tokenized while still providing the necessary features to train our ML models. Key mistakes include not properly masking data in the initial stages (Data Lake) or overprocessing the data too much, leading to loss of valuable information.

#### Common Mistakes/Follow-Ups
Over-Processing: Masking PII columns at multiple stages can introduce noise and reduce feature quality.
Inadequate Pipelines: Failing to implement real-time masking can lead to unmasked data exposure.
Compliance Violations: Not adhering strictly to regulatory requirements during the data processing pipeline can result in legal penalties.

**Comparison:**

| Step | Tool | Function |
|------|------|----------|
| Ingestion | Apache Beam, Spark Streaming | Masking PII Columns |
| Transformation | Airflow | Data Aggregation & Anonymization |
| Destination | ML Models | Usable Features |

**Code:**

```python
# Example code for real-time masking using Apache Beam
import apache_beam as beam

def mask_pii(record):
    # Implement PII masking logic here
    return record

with beam.Pipeline(options=pipeline_options) as p:
    raw_data = p | 'Read from Data Lake' >> ReadFromDataLake()
    masked_data = raw_data | 'Mask PII' >> beam.Map(mask_pii)
    _ = masked_data | 'Write to Data Lake' >> WriteToDataLake()
```

**Example input / output:**

| Input (Raw Data) | Masked Output |
|------------------|---------------|
| PII Column A: 123456, Feature B: 7.0 | PII Column A: 9876** , Feature B: 7.0 |

**Data:**

```json
{
  "data_lake": {
    "source": ["S3 Bucket"],
    "processing": [
      {"tool": "Apache Beam", "function": "Mask PII"},
      {"tool": "Spark Streaming", "function": "Real-time Data Ingestion"}
    ],
    "destination": ["Data Warehouse"]
  },
  "data_warehouse": {
    "source": ["Data Lake"],
    "processing": [
      {"tool": "Apache Spark", "function": "Aggregation & Anonymization"},
      {"tool": "Talend", "function": "ETL"}
    ],
    "destination": ["ML Models"]
  }
}
```

**Flow:**

1. Data Lake
2. Real-Time Masking
3. Masked Data
4. Data Warehouse
5. Transformed & AnonymizedFeatures for ML Models

