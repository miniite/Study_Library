---
type: "practice"
difficulty: "Intermediate"
---

# A COPY load succeeds but numbers are subtly wrong due to timezone/date parsing; how would you catch this class of bug systematically?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Time Travel and Storage Costs](<../../../../TOOLS/Snowflake/Snowflake_Platform/Time_Travel_and_Storage_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a COPY load in Snowflake succeeds, but the data is subtly incorrect due to timezone or date parsing issues, we need a systematic approach that includes logging and validation checks. This ensures we catch such bugs early in the pipeline.

First, let's understand what the question is probing: the COPY command in Snowflake can be prone to errors if date/time formats do not align with Snowflake’s expectations. These issues might go unnoticed until someone tries to query or use the data, leading to subtle but impactful bugs. The key is to implement robust logging and validation mechanisms.

#### Triaging the Question

#### Assumptions
We assume the COPY command is being used in a Data Engineering pipeline, where large volumes of data are regularly ingested.
What’s Being Probed: Robustness, error detection, logging practices, and data validation techniques.

#### Solution
from First Principles

Configure Logging:

Enable detailed logging for the COPY operation to capture any errors or warnings during ingestion. This can be done by setting up LOGERRORS option in the COPY statement.
SQL
COPY INTO table_name FROM 's3://path/to/csv' CREDENTIALS ...
    ON_ERROR = 'CONTINUE'
    LOG_ERRORS = true;

Validate Data After Ingestion:

Implement a post-processing step to validate the ingested data against expected formats and values.
Use Snowflake's SELECT statements with conditions that check for inconsistencies, such as invalid dates or timestamps.

Automate Validation Using Stored Procedures:

Write stored procedures in Snowflake to automate these validation checks.
SQL
CREATE OR REPLACE PROCEDURE validate_data()
RETURNS STRING
LANGUAGE SQL
AS
$$
    DECLARE
        result VARCHAR;
    BEGIN
        -- Check for invalid dates
        SELECT 'Invalid date found' INTO :result FROM table_name WHERE NOT DATE_FORMAT(column, '%Y-%m-%d') IS NOT NULL;

        IF LENGTH(:result) > 0 THEN
            RETURN :result;
        END IF;

        -- Additional validations can be added here

        RETURN 'Data validation passed';
    END;
$$;

CALL validate_data();

Monitor and Alert:

Set up monitoring and alerting for any errors or failed validations.
Use Snowflake’s built-in event monitors to trigger alerts when issues are detected.

#### Closing Summary

By configuring detailed logging, implementing automated data validation checks, and setting up robust monitoring, we can systematically catch issues like timezone/date parsing bugs early in the pipeline. This ensures that the ingested data is consistent with expectations, minimizing the risk of downstream errors or inconsistencies.

#### Common Mistakes/Follow-Ups

Mistake: Ignoring detailed logging during COPY operations.

#### Follow-Up
Always enable LOG_ERRORS to capture issues at the source.

Mistake: Not validating data post-ingestion.

#### Follow-Up
Regularly run validation checks and consider integrating them into CI/CD pipelines.

**Comparison:**

| Validations | Explanation |
|-------------|-------------|
| Date Format | Check if date is in proper format. |
| Timezone    | Ensure timezone is correct and consistent. |

**Code:**

```sql
-- Example of enabling logging during COPY operation
COPY INTO table_name FROM 's3://path/to/csv' CREDENTIALS ...
    ON_ERROR = 'CONTINUE'
    LOG_ERRORS = true;

-- Example of stored procedure for data validation
CREATE OR REPLACE PROCEDURE validate_data()
RETURNS STRING
LANGUAGE SQL
AS
$$
    DECLARE
        result VARCHAR;
    BEGIN
        -- Check for invalid dates
        SELECT 'Invalid date found' INTO :result FROM table_name WHERE NOT DATE_FORMAT(column, '%Y-%m-%d') IS NOT NULL;

        IF LENGTH(:result) > 0 THEN
            RETURN :result;
        END IF;

        RETURN 'Data validation passed';
    END;
$$;

-- Example of calling the validation procedure
CALL validate_data();
```

**Example input / output:**

```

Input:

SQL
Copy
COPY INTO orders FROM 's3://path/to/csv' CREDENTIALS ... ON_ERROR = 'CONTINUE' LOG_ERRORS = true;

Output (Log):

CODE
Copy
[INFO] COPY operation started for table_name.
[ERROR] Invalid date found in row: {"order_id": 123, "order_date": "invalid-date"}

Validation Call:

SQL
Copy
CALL validate_data();

Output (Procedure):

CODE
Copy
[INFO] Data validation passed.
```

**Data:**

```json
{
    "table_name": "orders",
    "column": "order_date"
}
```

**Flow:**

1. Start COPY Operation
2. Configure Logging
3. Ingest Data
4. Run Validation Checks
5. Alert on Issues Found
6. End Processing

