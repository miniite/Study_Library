---
type: "practice"
difficulty: "Deep"
---

# A source starts sending line-level records where it used to send order-level; your fact silently changes grain. How would you build a guardrail that alerts on this before dashboards go wrong?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of detecting a change in data granularity from order-level to line-level records, we need to ensure our data model and ETL processes are robust enough to catch discrepancies early on. In this scenario, we'll focus on building a guardrail using both data modeling techniques (Star/Snowflake Schema) and monitoring mechanisms.

First, let's summarize the key points:

The source system is now sending line-level records instead of order-level.
We need to detect this change before it affects our analytics dashboards.
We will use Star or Snowflake Schema to structure our data warehouse.
Monitoring and alerting mechanisms are crucial to catch anomalies.

Now, let's break down the problem step-by-step:

#### Triage

The question is essentially asking us how to design a monitoring system that can detect a change in data granularity from order-level to line-level records. We need to ensure that our ETL processes validate the incoming data against expected schemas and business logic.

#### Assumptions
The source of truth for orders (e.g., an order ID) is consistent across both types of records.
The target data warehouse uses a Star or Snowflake Schema.
We have access to historical data to compare current data with past patterns.
Step-by-Step Solution
1. Data Modeling

To handle the scenario, we'll use a combination of Star and Snowflake schemas:

placed
contains
sold
shipped
CUSTOMER
ORDER
LINE_ITEM
PRODUCT
SHIPMENT
Star Schema: We'll have a fact table LINE_ITEM that contains detailed records, and dimensions such as CUSTOMER, PRODUCT, and SHIPMENT.
Snowflake Schema: This involves normalizing the data further by splitting some of the dimensions into more granular tables (e.g., ORDER_DETAIL for order-level details).
2. ETL Process

The ETL process will be responsible for validating the incoming data against expected formats and business rules.

No
Yes
Data Source
Is Record Order-Level?
Validate Against KnownSchema
Alert on Granularity Change
Insert into LINE_ITEM FactTable
Validation Step: We'll implement a validation step to check if the incoming record is an order-level or line-level record. This can be done using schema checks, key existence validations, and logic-based rules.
Alerting Mechanism: If any discrepancies are detected (e.g., line-level records being inserted into the fact table meant for orders), we'll trigger an alert.
3. Monitoring and Alerting

We need to set up real-time monitoring to continuously check the incoming data against historical patterns:

Order-Level
Line-Level
Alert
Real-Time Data Ingestion
Validate Record Granularity
Insert into ORDER Fact Table
Validate Against KnownSchema
Send Alert to Monitoring Tool
Historical Data Comparison: We can use time-series analysis or machine learning models to detect anomalies by comparing incoming data against historical patterns.
Alerting Tools: Use tools like Prometheus, Grafana, or custom scripts to send alerts via email, Slack, or other monitoring systems.

#### Closing Summary

By using a combination of robust data modeling techniques and effective ETL processes with real-time validation and alerting mechanisms, we can ensure that our dashboards remain accurate even when there are changes in the source data granularity. This approach not only catches issues early but also ensures compliance with business rules and data quality standards.

#### Common Mistakes
Failing to implement proper validation checks during ETL.
Not setting up adequate monitoring and alerting mechanisms.
Ignoring historical data patterns, leading to delayed detection of anomalies.

