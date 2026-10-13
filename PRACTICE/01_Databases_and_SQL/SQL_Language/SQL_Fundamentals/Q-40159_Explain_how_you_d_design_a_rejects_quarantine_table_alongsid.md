---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd design a rejects/quarantine table alongside a clean target so bad rows are captured with a rejection reason, and query the daily reject rate per rule to trend data quality.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a system that captures bad rows (rejection reasons) alongside a clean target table, we need to ensure robustness in capturing issues and providing actionable insights into data quality trends. The key idea is to create a "rejects" or "quarantine" table that logs each rejected row along with the reason for rejection. Additionally, we'll implement a process to calculate the daily reject rate per rule to monitor data integrity over time.

#### Triaging the Question

The question is essentially asking how to design and manage two tables: one clean target table where good data resides, and another rejects/quarantine table that logs bad rows along with rejection reasons. The primary goal is to track the number of rejected rows daily for each rule used in validation. This helps in understanding data quality issues over time.

#### First-Principles Solution

Schema Design:

Clean Target Table: Stores clean, validated records.
Rejects/Quarantine Table: Logs rejected records with detailed reasons.

Data Validation Rules:

Define a set of validation rules that check for common issues such as missing values, incorrect formats, and outliers.

Insertion Process:

For each incoming record, apply the defined validation rules.
If a record fails any rule, log it in the rejects table with the specific reason(s) for rejection.

Daily Reject Rate Calculation:

Use SQL queries to aggregate the number of rejected rows daily by each rule and calculate the reject rate.

Monitoring & Reporting:

Implement periodic reports to track trends over time, helping stakeholders understand data quality issues.
Detailed Steps

Schema Design

SQL
-- Clean Target Table
CREATE TABLE clean_data (
    id SERIAL PRIMARY KEY,
    [other fields] VARCHAR(255),
    [insert other relevant columns]
);

-- Rejects/Quarantine Table
CREATE TABLE rejects (
    id SERIAL PRIMARY KEY,
    clean_data_id INT REFERENCES clean_data(id) ON DELETE CASCADE,
    rule_name VARCHAR(100),  -- e.g., 'missing_value', 'invalid_format'
    rejection_reason TEXT,   -- Detailed reason for the rejection
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

Data Validation Rules

SQL
-- Example validation rules:
CREATE OR REPLACE FUNCTION validate_data()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.[some_field] IS NULL THEN
        INSERT INTO rejects (clean_data_id, rule_name, rejection_reason) VALUES (NEW.id, 'missing_value', 'Field is null');
    ELSIF NOT REGEXP_LIKE(NEW.[another_field], '^[a-zA-Z0-9]+$') THEN
        INSERT INTO rejects (clean_data_id, rule_name, rejection_reason) VALUES (NEW.id, 'invalid_format', 'Invalid format for [another_field]');
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to apply validation rules on insert
CREATE TRIGGER validate_trigger BEFORE INSERT ON clean_data FOR EACH ROW EXECUTE FUNCTION validate_data();

Daily Reject Rate Calculation

SQL
SELECT
    rule_name,
    COUNT(*) as reject_count,
    COUNT(*) / (SELECT COUNT(*) FROM clean_data WHERE timestamp >= CURRENT_DATE - INTERVAL '1 day') * 100 AS reject_rate_percent
FROM
    rejects
WHERE
    timestamp >= CURRENT_DATE - INTERVAL '1 day'
GROUP BY
    rule_name;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key is to ensure robust schema design, proper validation rules, and efficient daily reject rate calculation. Common mistakes include missing detailed rejection reasons or not cascading deletions properly in case of data cleansing.

#### Follow-ups
could involve discussing how to handle concurrent insertions, ensuring data integrity during migrations, or integrating this system with existing ETL pipelines for seamless operations.

**Comparison:**

```
has
CLEAN_DATA
int
id
PK
varchar
some_field
text
another_field
REJECTS
int
id
PK
int
clean_data_id
FK
varchar
rule_name
text
rejection_reason
timestamp
timestamp
rejected
rows
```

**Code:**

```sql
-- Clean Target Table Schema
CREATE TABLE clean_data (
    id SERIAL PRIMARY KEY,
    some_field VARCHAR(255),
    another_field TEXT,
    [insert other relevant columns]
);

-- Rejects/Quarantine Table Schema
CREATE TABLE rejects (
    id SERIAL PRIMARY KEY,
    clean_data_id INT REFERENCES clean_data(id) ON DELETE CASCADE,
    rule_name VARCHAR(100),  -- e.g., 'missing_value', 'invalid_format'
    rejection_reason TEXT,   -- Detailed reason for the rejection
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Validation Function
CREATE OR REPLACE FUNCTION validate_data()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.some_field IS NULL THEN
        INSERT INTO rejects (clean_data_id, rule_name, rejection_reason) VALUES (NEW.id, 'missing_value', 'Field is null');
    ELSIF NOT REGEXP_LIKE(NEW.another_field, '^[a-zA-Z09]+$') THEN
        INSERT INTO rejects (clean_data_id, rule_name, rejection_reason) VALUES (NEW.id, 'invalid_format', 'Invalid format for another_field');
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Trigger to apply validation rules on insert
CREATE TRIGGER validate_trigger BEFORE INSERT ON clean_data FOR EACH ROW EXECUTE FUNCTION validate_data();

-- Daily Reject Rate Calculation Query
SELECT
    rule_name,
    COUNT(*) as reject_count,
    COUNT(*) / (SELECT COUNT(*) FROM clean_data WHERE timestamp >= CURRENT_DATE - INTERVAL '1 day') * 100 AS reject_rate_percent
FROM
    rejects
WHERE
    timestamp >= CURRENT_DATE - INTERVAL '1 day'
GROUP BY
    rule_name;
```

**Example input / output:**

```text
-- Inserting a clean data record
INSERT INTO clean_data (some_field, another_field) VALUES ('value', 'good_value');

-- Query to calculate daily reject rate
SELECT
    rule_name,
    COUNT(*) as reject_count,
    COUNT(*) / (SELECT COUNT(*) FROM clean_data WHERE timestamp >= CURRENT_DATE - INTERVAL '1 day') * 100 AS reject_rate_percent
FROM
    rejects
WHERE
    timestamp >= CURRENT_DATE - INTERVAL '1 day'
GROUP BY
    rule_name;
```

**Data:**

```json
{
    "clean_data": [
        {
            "id": 1,
            "some_field": "value",
            "another_field": "good_value"
        },
        {
            "id": 2,
            "some_field": NULL,
            "another_field": ""
        }
    ],
    "rejects": [
        {
            "id": 1,
            "clean_data_id": 2,
            "rule_name": "missing_value",
            "rejection_reason": "Field is null"
        },
        {
            "id": 2,
            "clean_data_id": 2,
            "rule_name": "invalid_format",
            "rejection_reason": "Invalid format for another_field"
        }
    ]
}
```

**Flow:**

1. Incoming Record
2. Validation Rules
3. Record passes validation,insert into CLEAN_DATA
4. Record fails validation, log inREJECTS and insert intoCLEAN_DATA
5. Calculate daily reject rate perrule

