---
type: "practice"
difficulty: "Expert"
---

# How would you architect end-to-end column-level and row-level data lineage across ingestion, transformation, and serving so an auditor can trace any figure back to source?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Real-time Features](<../../../../05_Data_Governance_and_Security/Governance/Stewardship/Governing_Real-time_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect end-to-end column-level and row-level data lineage in a database system, you need to ensure that every piece of data has a clear provenance trail from its initial ingestion through all transformations to eventual serving or storage. This involves designing a robust metadata management framework, implementing appropriate tracking mechanisms at each step, and ensuring these mechanisms are integrated seamlessly.

#### Triage

The core challenge is to create a system where the origin of any data point can be traced back to its source, along with all modifications made during processing. The interviewer is probing your understanding of:

Metadata management principles.
Data lineage tracking techniques.
Integration complexities across ingestion, transformation, and serving layers.

#### First-Principles Solution
Metadata Schema Design: Establish a metadata schema that captures essential details such as the data element (column), its source system, timestamp of processing steps, and the nature of transformations applied.
Ingestion Layer: Ensure each piece of ingested data is tagged with its origin and initial state.
Transformation Layer: Implement logging mechanisms to capture changes in individual columns or rows during ETL processes. Use database triggers or custom scripts for this purpose.
Serving Layer: Maintain lineage by storing metadata alongside the transformed data, possibly through a separate audit table or using version control systems like Git.
Detailed Solution

Metadata Schema

Define a DataLineage entity with fields: data_id, origin_system, origin_timestamp, processed_by, processing_time, transformations_applied, etc.

Ingestion Layer

SQL
CREATE TABLE Orders (
    order_id INT,
    customer_name VARCHAR(100),
    product_name VARCHAR(50),
    quantity INT,
    order_date TIMESTAMP
);
Tag each row with metadata: INSERT INTO Orders (order_id, customer_name, ..., origin_system, origin_timestamp) VALUES (?, ?, ..., 'source', NOW()).

Transformation Layer

SQL
CREATE TABLE ProcessedOrders (
    order_id INT,
    customer_name VARCHAR(100),
    product_name VARCHAR(50),
    quantity INT,
    order_date TIMESTAMP,
    transformation_steps TEXT,
    processed_by VARCHAR(255)
);
Implement triggers or custom scripts to log changes:
SQL
CREATE TRIGGER LogTransformations AFTER UPDATE ON Orders FOR EACH ROW
BEGIN
    INSERT INTO DataLineage (data_id, origin_system, origin_timestamp, transformations_applied, processed_by)
    VALUES (?, 'source', NOW(), JSON_ARRAY_APPEND('[]', JSON_OBJECT('column', column_name, 'value', NEW.column_value), 'transformation details'), 'processor');
END;

Serving Layer

Serve data with its lineage:
SQL
SELECT order_id, customer_name, product_name, quantity, order_date,
       JSON_OBJECTAGG(transformations_applied, processed_by) AS lineage_data
FROM ProcessedOrders GROUP BY order_id;

#### Closing Summary
+ Common Mistakes/Follow-Ups

This architecture ensures comprehensive data lineage by tracking changes at the column and row level throughout the entire lifecycle of a data asset. Key areas to focus on include:

#### Performance Considerations
Ensure metadata logging does not impact performance significantly.
Scalability: Design the system to handle large volumes of data efficiently, possibly using distributed systems or microservices architectures.
Security & Compliance: Securely manage access to lineage data and ensure it complies with organizational policies.

#### Common Mistakes
Over-engineering: Complex metadata schemes that are difficult to maintain.
Ignoring performance implications of additional logging.
Failing to integrate transformation details properly, leading to gaps in the lineage trail.

**Comparison:**

| order_id | customer_name   | product_name   | quantity | order_date           |
|----------|-----------------|---------------|---------|---------------------|
| 123456   | john_doe        | apple_product  | 10      | 2023-09-15 10:00:00Z |
| dataLineage |
|-------------|
| {           |
|     "dataId": 123456,          |
|     "originSystem": "source",   |
|     "originTimestamp": "2023-09-15T10:00:00Z", |
|     "transformationsApplied": [ |
|       {                         |
|         "column": "customer_name", |
|         "value": "John Doe",      |
|         "action": "lowercase"     |
|       },                         |
|       {                         |
|         "column": "product_name", |
|         "value": "Apple Product", |
|         "action": "replace(' ', '_')"  |
|       }                          |
|     ],                          |
|     "processedBy": "processorA"   |
| }                               |

**Code:**

```sql
-- Example SQL Triggers and Procedures

CREATE TABLE Orders (
    order_id INT,
    customer_name VARCHAR(100),
    product_name VARCHAR(50),
    quantity INT,
    order_date TIMESTAMP
);

INSERT INTO Orders (order_id, customer_name, ..., origin_system, origin_timestamp) VALUES (?, ?, ..., 'source', NOW());

CREATE TABLE ProcessedOrders (
    order_id INT,
    customer_name VARCHAR(100),
    product_name VARCHAR(50),
    quantity INT,
    order_date TIMESTAMP,
    transformation_steps TEXT,
    processed_by VARCHAR(255)
);

CREATE TRIGGER LogTransformations AFTER UPDATE ON Orders FOR EACH ROW
BEGIN
    INSERT INTO DataLineage (data_id, origin_system, origin_timestamp, transformations_applied, processed_by)
    VALUES (?, 'source', NOW(), JSON_ARRAY_APPEND('[]', JSON_OBJECT('column', column_name, 'value', NEW.column_value), 'transformation details'), 'processor');
END;

SELECT order_id, customer_name, product_name, quantity, order_date,
       JSON_OBJECTAGG(transformations_applied, processed_by) AS lineage_data
FROM ProcessedOrders GROUP BY order_id;
```

**Example input / output:**

```text
Input:
INSERT INTO Orders (order_id, customer_name, product_name, quantity) VALUES (123456, 'John Doe', 'Apple Product', 10);

Output:
(123456, 'john_doe', 'apple_product', 10, '2023-09-15 10:00:00')

Metadata Log:
{
  "dataId": 123456,
  "originSystem": "source",
  "originTimestamp": "2023-09-15T10:00:00Z",
  "transformationsApplied": [
    {
      "column": "customer_name",
      "value": "John Doe",
      "action": "lowercase"
    },
    {
      "column": "product_name",
      "value": "Apple Product",
      "action": "replace(' ', '_')"
    }
  ],
  "processedBy": "processorA"
}
```

**Data:**

```json
{
  "dataLineage": [
    {
      "dataId": 123456,
      "originSystem": "source",
      "originTimestamp": "2023-09-15T10:00:00Z",
      "transformationsApplied": ["lowercase(customer_name)", "replace(product_name, ' ', '_')"],
      "processedBy": "processorA"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Ingestion
4. Is Data Altered?
5. Logging Alterations
6. Serving
7. Auditing Lineage

