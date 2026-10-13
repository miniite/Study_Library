---
type: "practice"
difficulty: "Expert"
---

# How do you handle enterprise master-data management so a conformed customer dimension reflects a single golden record across CRM, billing, and support?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with enterprise master-data management in a complex environment like CRM, billing, and support systems, the goal is to ensure that a conformed customer dimension reflects a single "golden record." This involves integrating data from multiple sources into a consistent, reliable model while maintaining historical accuracy. The star or snowflake schema can be effectively employed here to achieve this objective.

Firstly, it's important to understand that we are dealing with data warehousing principles where the primary focus is on analytics and reporting rather than transactional processing. The architecture will involve ensuring data consistency, integrity, and completeness across different source systems.

The key steps include:

Identifying Data Sources: Determine which systems (CRM, billing, support) need to be integrated.
Data Extraction & Transformation: Use ETL processes to extract relevant customer data from each system and transform it into a consistent format.
Staging Area: Store transformed data in a staging area before loading it into the warehouse.
Conformed Dimensions Table: Create a single, unified conformed dimension table for customers that serves as the central record.

Let's break this down step by step:

#### Triage

The question is asking how to handle master-data management across multiple systems so that customer data can be aggregated and presented with a single golden record. The interviewer might be probing your understanding of:

Data integration techniques
ETL processes
Data warehousing concepts like star and snowflake schemas
Historical data retention and consistency

#### Assumptions
and Context
Source Systems: CRM, billing, support systems are the primary sources.
Data Volume: Significant volume with high transaction rates from each system.
Historical Data: Maintaining historical records for audit purposes.
Step-by-Step Solution
1. Identify Data Sources

Assume we have three main source systems:

CRM: Customer interactions, including sales, marketing, and support tickets.
Billing System: Financial transactions associated with customers.
Support System: Customer service interactions and issues reported by users.

Each system has its own customer record structure, but they share common attributes like customer_id, name, email, etc.

2. Data Extraction & Transformation

For each source system, the ETL process will:

Extract data from the source systems.
Transform the data to a consistent format (e.g., standardizing names and email formats).
Apply business rules to clean and enrich the data.

For example, in SQL:

SQL
-- CRM System ETL Transformation Example
INSERT INTO Staging.CRM_Customer (
    customer_id,
    name,
    email,
    phone_number,
    first_contact_date,
    last_contact_date
)
SELECT
    c.customer_id,
    UPPER(c.name) AS name,
    LOWER(c.email) AS email,
    REGEXP_REPLACE(c.phone_number, '[^0-9]', '') AS phone_number,
    c.first_contact_date,
    c.last_contact_date
FROM CRM.customers c;

Similar transformations are applied to the billing and support systems.

3. Staging Area

The transformed data is stored in a staging area table for further processing:

SQL
CREATE TABLE Staging.CRM_Customer (
    customer_id INT,
    name VARCHAR(255),
    email VARCHAR(255),
    phone_number VARCHAR(15),
    first_contact_date DATE,
    last_contact_date DATE
);
4. Conformed Dimensions Table

Create a conformed dimensions table that will serve as the single source of truth for customer data:

SQL
CREATE TABLE Warehouse.Customer_Dimensions (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    phone_number VARCHAR(15),
    first_contact_date DATE,
    last_contact_date DATE,
    is_golden_record BOOLEAN DEFAULT FALSE,
    golden_record_source VARCHAR(50)
);

During the ETL process, each record from the staging area is evaluated and marked as a potential golden record. The golden_record_source field indicates which system originally provided this data.

5. Data Loading & Synchronization

After all ETL processes complete, use batch or real-time processes to load and update the conformed dimensions table:

SQL
-- Batch Update Example
MERGE INTO Warehouse.Customer_Dimensions AS target
USING Staging.CRM_Customer AS source
ON (target.customer_id = source.customer_id)
WHEN MATCHED THEN
    UPDATE SET
        name = source.name,
        email = source.email,
        phone_number = source.phone_number,
        first_contact_date = source.first_contact_date,
        last_contact_date = source.last_contact_date,
        golden_record_source = 'CRM'
WHERE source.is_golden_record;

-- Real-Time Update Example
CREATE OR REPLACE TRIGGER update_customer_dimension
AFTER INSERT ON Staging.CRM_Customer FOR EACH ROW
BEGIN
    UPDATE Warehouse.Customer_Dimensions
    SET
        name = :NEW.name,
        email = :NEW.email,
        phone_number = :NEW.phone_number,
        first_contact_date = :NEW.first_contact_date,
        last_contact_date = :NEW.last_contact_date,
        golden_record_source = 'CRM'
    WHERE customer_id = :NEW.customer_id
    AND NOT is_golden_record;
END;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the approach involves creating a staging area, transforming data from multiple sources, identifying a single golden record for each customer across all systems, and maintaining historical records. This ensures consistency and integrity in master-data management.

#### Common mistakes
to avoid include:

Failing to use ETL processes, which can lead to inconsistent or inaccurate data.
Not maintaining historical records, leading to loss of audit trails.
Overlooking potential conflicts when updating the conformed dimensions table.

#### Follow-up
questions might involve:

How would you handle updates in real-time?
What performance optimizations could be applied to large datasets?

**Comparison:**

```
tableDiagram
    direction LR
    CRM_system -->|ETL| Staging_area
    Staging_area --> Warehouse_customer_dimensions
    Support_system -->|ETL| Staging_area
    Billing_system -->|ETL| Staging_area
    Staging_area -->|Batch/Real-time updates| Warehouse_customer_dimensions
```

**Code:**

```sql
-- ETL Transformation Example (CRM)
INSERT INTO Staging.CRM_Customer (
    customer_id,
    name,
    email,
    phone_number,
    first_contact_date,
    last_contact_date
)
SELECT
    c.customer_id,
    UPPER(c.name) AS name,
    LOWER(c.email) AS email,
    REGEXP_REPLACE(c.phone_number, '[^0-9]', '') AS phone_number,
    c.first_contact_date,
    c.last_contact_date
FROM CRM.customers c;

-- Conformed Dimensions Table Creation
CREATE TABLE Warehouse.Customer_Dimensions (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    phone_number VARCHAR(15),
    first_contact_date DATE,
    last_contact_date DATE,
    is_golden_record BOOLEAN DEFAULT FALSE,
    golden_record_source VARCHAR(50)
);

-- Batch Update Example
MERGE INTO Warehouse.Customer_Dimensions AS target
USING Staging.CRM_Customer AS source
ON (target.customer_id = source.customer_id)
WHEN MATCHED THEN
    UPDATE SET
        name = source.name,
        email = source.email,
        phone_number = source.phone_number,
        first_contact_date = source.first_contact_date,
        last_contact_date = source.last_contact_date,
        golden_record_source = 'CRM'
WHERE source.is_golden_record;

-- Real-Time Update Example
CREATE OR REPLACE TRIGGER update_customer_dimension
AFTER INSERT ON Staging.CRM_Customer FOR EACH ROW
BEGIN
    UPDATE Warehouse.Customer_Dimensions
    SET
        name = :NEW.name,
        email = :NEW.email,
        phone_number = :NEW.phone_number,
        first_contact_date = :NEW.first_contact_date,
        last_contact_date = :NEW.last_contact_date,
        golden_record_source = 'CRM'
    WHERE customer_id = :NEW.customer_id
    AND NOT is_golden_record;
END;
```

**Example input / output:**

```text
-- Input (Example Staging Data)
INSERT INTO Staging.CRM_Customer (
    customer_id,
    name,
    email,
    phone_number,
    first_contact_date,
    last_contact_date
) VALUES
(1, 'John Doe', 'johndoe@example.com', '+1234567890', '2023-01-01', '2023-01-31');

-- Output (Updated Customer Dimension)
UPDATE Warehouse.Customer_Dimensions
SET
    name = 'John Doe',
    email = 'johndoe@example.com',
    phone_number = '+1234567890',
    first_contact_date = '2023-01-01',
    last_contact_date = '2023-01-31',
    golden_record_source = 'CRM'
WHERE customer_id = 1 AND NOT is_golden_record;
```

**Data:**

```sql
CREATE TABLE Staging.CRM_Customer (
    customer_id INT,
    name VARCHAR(255),
    email VARCHAR(255),
    phone_number VARCHAR(15),
    first_contact_date DATE,
    last_contact_date DATE
);

CREATE TABLE Warehouse.Customer_Dimensions (
    customer_id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255),
    phone_number VARCHAR(15),
    first_contact_date DATE,
    last_contact_date DATE,
    is_golden_record BOOLEAN DEFAULT FALSE,
    golden_record_source VARCHAR(50)
);
```

**Flow:**

1. Extract
2. Transform
3. Extract
4. Extract
5. CRM System
6. Staging Area
7. Warehouse CustomerDimensions
8. Support System
9. Billing System
10. Batch/Real-time Updates

