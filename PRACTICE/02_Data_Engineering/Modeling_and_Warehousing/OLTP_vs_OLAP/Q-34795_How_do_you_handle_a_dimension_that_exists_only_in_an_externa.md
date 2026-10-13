---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle a dimension that exists only in an external SaaS (not the OLTP DB) but must join to OLTP-sourced facts in the warehouse — where does that data land and how fresh must it be?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Separating OLTP and Reporting](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Separating_OLTP_and_Reporting.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a dimension existing only in an external SaaS, we need to ensure that this dimension is properly integrated into our data warehouse while maintaining its freshness and consistency with the OLTP sources. The key challenge here is ensuring that the external data can be reliably joined with internal OLTP facts without disrupting the overall ETL (Extract, Transform, Load) processes.

Firstly, let’s break down what we’re dealing with:

Dimension from SaaS: This dimension is sourced externally and does not exist in our OLTP database.
OLTP Facts: These are transactional data that will be loaded into the data warehouse for analytics purposes.
Freshness Requirement: The external dimension must be kept up-to-date to ensure accurate joins with internal facts.

Now, let’s delve into a step-by-step solution:

Step 1: ETL Design

The first step is to design an effective ETL process that can periodically refresh the dimension data from the SaaS. This can be done using scheduled tasks such as Airflow DAGs or custom scripts.

#### Example
Workflow:
fails
Fetch External Data
Load into Staging Table
Transform and Cleanse
Upload to WarehouseDimension Table
Error Handling
Step 2: Data Storage

Store the external dimension data in a staging table that is separate from your main dimension table. This allows for incremental updates without affecting ongoing queries.

#### Example
Schema:
SQL
CREATE TABLE SaaS_Dimension (
    DimensionKey INT PRIMARY KEY,
    Attribute1 VARCHAR(50),
    Attribute2 VARCHAR(50)
);
Step 3: Integration with OLTP Facts

When joining the external dimension with OLTP facts in the data warehouse, ensure that your ETL process handles this join correctly. This might involve writing custom SQL to perform a left join or using a transformation step within the ETL pipeline.

#### Example
Join:
SQL
SELECT
    F.*,
    D.Attribute1,
    D.Attribute2
FROM
    OLTP_Facts F
LEFT JOIN
    SaaS_Dimension D ON F.DimensionKey = D.DimensionKey;
Step 4: Ensuring Freshness

Ensure that the external data is refreshed at a cadence sufficient for your analytical needs. For instance, if real-time analysis isn't required, daily or hourly refreshes might be appropriate.

#### Example
Refresh Strategy:
Daily Batch: If the SaaS API supports scheduled refreshes, set up a task to fetch and update the staging table every day.
Real-Time Triggered: If the SaaS offers an event-driven API, use it to trigger updates immediately when changes are detected.
Step 5: Monitoring and Validation

Implement monitoring to ensure that the ETL process is running smoothly and that the data in the dimension table is accurate. This includes setting up alerts for failures and regular audits of the freshness and integrity of the external data.

#### Example
Alerts:
ETL Failure Alert: If a scheduled task fails, notify the team via email or Slack.
Data Quality Check: Run periodic queries to verify that new data has been correctly imported and joined.

#### Closing Summary

In summary, handling an external dimension requires careful planning of ETL processes, proper data storage, integration with OLTP facts, ensuring data freshness, and robust monitoring. The key is to maintain a balance between the reliability of the process and the timeliness of the data.

#### Common Mistakes

Ignoring Data Quality: Not validating the imported data can lead to incorrect analytics.
Insufficient Monitoring: Missing critical errors or issues with the ETL process can cause data integrity problems.
Inadequate Synchronization: Not ensuring that external and internal data are in sync can result in inconsistent joins.

**Comparison:**

| Step | Action              | Notes                          |
|------|---------------------|-------------------------------|
| 1    | Fetch External Data | Use API or scheduled task      |
| 2    | Load into Staging   | Validate and cleanse data      |
| 3    | Transform and Cleanse | Standardize formats           |
| 4    | Upload to Warehouse Dimension Table | Ensure integrity checks        |

**Code:**

```sql
CREATE TABLE SaaS_Dimension (
    DimensionKey INT PRIMARY KEY,
    Attribute1 VARCHAR(50),
    Attribute2 VARCHAR(50)
);

-- Example ETL Step to Load Data
INSERT INTO SaaS_Dimension (DimensionKey, Attribute1, Attribute2)
SELECT * FROM ExternalSaaSAPI('https://api.example.com/dimensions');

-- Example Join in Warehouse
SELECT
    F.*,
    D.Attribute1,
    D.Attribute2
FROM
    OLTP_Facts F
LEFT JOIN
    SaaS_Dimension D ON F.DimensionKey = D.DimensionKey;
```

**Example input / output:**

```text
-- Staging Table Initial State
SaaS_Dimension:
| DimensionKey | Attribute1   | Attribute2   |
|--------------|--------------|--------------|
|            1 | DimValue1    | DimValue2    |

-- After ETL Process
SaaS_Dimension:
| DimensionKey | Attribute1   | Attribute2   |
|--------------|--------------|--------------|
|            1 | UpdatedValueA | UpdatedValueB |
```

**Data:**

```json
{
  "DimensionKey": 101,
  "Attribute1": "DimValueA",
  "Attribute2": "DimValueB"
}
```

**Flow:**

1. Fetch External Data
2. Load into Staging Table
3. Transform and Cleanse
4. Upload to WarehouseDimension Table
5. Monitor and Validate
6. Freshness Check Alert

