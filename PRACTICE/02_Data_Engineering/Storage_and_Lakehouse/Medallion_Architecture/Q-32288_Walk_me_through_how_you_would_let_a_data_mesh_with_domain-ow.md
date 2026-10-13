---
type: "practice"
difficulty: "Expert"
---

# Walk me through how you would let a data mesh with domain-owned products coexist with a central medallion lakehouse without recreating a monolithic warehouse.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Controlling Gold Sprawl](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Controlling_Gold_Sprawl.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a data mesh with domain-owned products can coexist with a central medallion lakehouse in a harmonious manner, we need to design a solution that leverages the strengths of both architectures while mitigating their inherent trade-offs. The core idea is to create an ecosystem where the domains maintain control over their own data but still benefit from a centralized layer for data transformation and governance.

First, let's break down the key components and principles involved:

Data Mesh: This architecture emphasizes domain ownership of data assets and promotes autonomous teams.
Medallion Architecture: It typically consists of three layers: Bronze (raw), Silver (transformed), and Gold (aggregated).

The challenge is to integrate these two architectures without overcomplicating the system or duplicating efforts.

#### Triaging the Question
The interviewer is probing how to maintain autonomy in data management while ensuring that a central layer for critical transformations exists.
We need to consider assumptions such as the existing infrastructure, governance requirements, and technical feasibility.
The goal is to design an architecture where domains can operate independently but still benefit from centralized processes.
Step-by-Step Solution

Define Data Domains: Identify key business domains that will own their data. For instance, domains might include Sales, Marketing, Finance, etc.

Domain-Specific Storage: Each domain stores its raw and transformed data in its respective warehouse (e.g., BigQuery, Redshift). These warehouses are the foundation of a Data Mesh.

Medallion Lakehouse: Create a central medallion lakehouse that serves as a transformation layer. This can be implemented using tools like Delta Lake or Apache Iceberg for scalable and reliable data storage.

Data Pipelines and APIs:

Domains can write their raw data to their local warehouses.
A set of ETL (Extract, Transform, Load) jobs can push transformed data from the domain-specific warehouses into the central medallion lakehouse.
APIs can be created for domains to read or query data directly from both their own and the central storage.

Governance and Access: Implement a governance layer that ensures compliance with organizational policies across all data sources. This could involve creating roles, permissions, and auditing mechanisms.

Monitoring and Alerting: Set up monitoring and alerting for performance and integrity of the system to ensure that both the domain-specific systems and the central lakehouse are functioning as expected.

#### Closing Summary

By combining a Data Mesh with a Medallion Architecture, we can achieve a balance between decentralized data ownership and centralized governance. This approach allows domains to maintain control over their data while still benefiting from a unified layer for critical transformations and analytics.

#### Common Mistakes/Follow-Ups

Mistake: Overlapping responsibilities leading to inefficiencies or conflicts.

#### Follow-Up
Clearly define roles, responsibilities, and communication channels between the domains and the central governance team.

Mistake: Underestimating data lineage and traceability.

#### Follow-Up
Ensure robust metadata management and lineage tracking to maintain transparency in data transformations.

**Comparison:**

| Domain   | Warehouse Type | Tables               |
|----------|----------------|----------------------|
| Sales    | BigQuery       | bigquery:sales       |
| Marketing| Redshift       | redshift:marketing  |
| ...      |                |                      |
| Medallion Lakehouse | Layer           | Table                  |
|---------------------|-----------------|------------------------|
| Bronze              | Delta Lake      | bronze                 |
| Silver              | Delta Lake      | silver                 |
| Gold                | Delta Lake      | gold                   |

**Code:**

```python
# Example ETL job for transforming data from a domain-specific warehouse to the medallion lakehouse

def transform_data(source_table, target_table):
    # Load raw data from source table
    raw_data = spark.read.table(source_table)

    # Perform transformations (e.g., cleaning, aggregations)
    transformed_data = raw_data.transform(clean_data).agg(...)

    # Write transformed data to the medallion lakehouse
    transformed_data.write.mode('append').format("delta").saveAsTable(target_table)
```

**Example input / output:**

```text
Input (Domain-Specific Warehouse):
- Sales warehouse writes raw data to bigquery:sales

Output (Medallion Lakehouse):
- Raw data is transformed and written to delta lake:bronze
- Bronze layer data is further transformed and written to delta lake:silver
- Silver layer data is aggregated and stored in delta lake:gold
```

**Data:**

```json
{
  "domains": [
    {"name": "Sales", "warehouse": "bigquery:sales"},
    {"name": "Marketing", "warehouse": "redshift:marketing"},
    ...
  ],
  "medallion_lakehouse": {
    "bronze_layer": "delta lake:bronze",
    "silver_layer": "delta lake:silver",
    "gold_layer": "delta lake:gold"
  }
}
```

**Flow:**

1. Yes
2. No
3. Medallion Lakehouse
4. Bronze Layer
5. Silver Layer
6. Gold Layer
7. Domain Warehouse
8. ETL Job
9. Transformed Data
10. MEDALLION LAKEHOUSE
11. Audit Logs
12. Governance Checks

