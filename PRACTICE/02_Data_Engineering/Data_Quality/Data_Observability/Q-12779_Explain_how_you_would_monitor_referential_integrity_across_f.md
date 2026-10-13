---
type: "practice"
difficulty: "Deep"
---

# Explain how you would monitor referential integrity across fact and dimension tables in a warehouse without primary key enforcement.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Quality Monitoring](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Data_Quality_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure referential integrity (RI) across fact and dimension tables in a data warehouse without relying on primary keys, we need to leverage a combination of metadata, constraints, and operational monitoring techniques. This approach involves setting up comprehensive checks that enforce RI through secondary key relationships, surrogate keys, or even logical constraints. The goal is to ensure data consistency, detect anomalies early, and prevent cascading errors.

#### Triaging the Question

The core issue here is ensuring referential integrity in a data warehouse without primary keys. The interviewer is probing how you would design a system that can monitor these relationships effectively. Key assumptions include:

The presence of surrogate or natural keys in dimension tables.
Potential use of transactional metadata for tracking changes.
Availability of a robust monitoring framework.

#### First Principles Solution
Define RI Constraints Using Surrogate Keys:
Even without primary keys, ensure that each dimension table has a unique identifier (surrogate key) and uses this to establish relationships with fact tables.
Implement Metadata Tracking:
Track changes in both dimensions and facts using metadata logs or audit trails.
Periodic and Real-Time Validation:
Develop scripts or jobs to periodically validate RI constraints.
Event-Driven Monitoring:
Use event-driven architectures (EDA) to trigger validation checks whenever there are updates in the database.
Step-by-Step Solution
1. Define Surrogate Keys

In dimension tables, surrogate keys can be used instead of natural keys for establishing relationships with fact tables. For instance:

SQL
CREATE TABLE DimCustomers (
    CustomerID INT PRIMARY KEY,
    Name VARCHAR(255),
    Email VARCHAR(255)
);

And in the fact table:

SQL
CREATE TABLE FactSales (
    SaleID INT PRIMARY KEY,
    Date DATE,
    Amount DECIMAL(10, 2),
    CustomerID INT,
    FOREIGN KEY (CustomerID) REFERENCES DimCustomers(CustomerID)
);
2. Implement Metadata Tracking

Use a metadata schema to track changes in the data:

SQL
CREATE TABLE AuditLog (
    LogID INT PRIMARY KEY AUTO_INCREMENT,
    TableName VARCHAR(100),
    RecordID INT,
    OperationType ENUM('INSERT', 'UPDATE', 'DELETE'),
    Timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

When an update occurs, log it to this table.

3. Periodic Validation Script

Create a periodic validation script that checks RI constraints:

SQL
-- Example: Validate CustomerID in FactSales against DimCustomers
SELECT fs.SaleID, dc.CustomerID, COUNT(*)
FROM FactSales fs
LEFT JOIN DimCustomers dc ON fs.CustomerID = dc.CustomerID
GROUP BY fs.SaleID, dc.CustomerID
HAVING COUNT(*) > 1;
4. Event-Driven Monitoring

Set up triggers or hooks to detect changes and trigger validation:

SQL
CREATE TRIGGER trg_FactSales_ValidateRI
AFTER INSERT ON FactSales
FOR EACH ROW
BEGIN
    -- Trigger RI validation logic here
END;

#### Closing Summary
and Common Mistakes

To summarize, ensuring referential integrity in a data warehouse without primary keys involves using surrogate keys, implementing metadata tracking, setting up periodic checks, and leveraging event-driven monitoring. Common mistakes include neglecting to track changes effectively or not having robust validation mechanisms in place.

Common Follow-Ups:

How would you handle RI when dealing with large datasets?
What are the trade-offs between using surrogate vs. natural keys for RI enforcement?
How do you ensure that your validation logic is optimized for performance?

