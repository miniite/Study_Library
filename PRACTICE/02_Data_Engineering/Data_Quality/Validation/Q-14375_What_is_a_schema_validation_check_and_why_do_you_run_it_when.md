---
type: "practice"
difficulty: "Basic"
---

# What is a schema validation check and why do you run it when ingesting data?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Quality Fundamentals](<../../../../02_Data_Engineering/Data_Quality/Validation/Data_Quality_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Schema validation checks are essential procedures in ensuring that the data being ingested into a system meets specific predefined rules or structures. Essentially, they help maintain the integrity of your datasets by verifying that each piece of incoming data is correctly formatted and adheres to the expected schema.

Firstly, let's break down what we're talking about:

What is Schema Validation Check? - It’s a process where the structure and type of incoming data are compared against a predefined schema. A schema defines the format and constraints that the data should follow.
Why Run It? - Running these checks ensures that only valid, structured data enters your system, which prevents issues downstream such as data corruption, misinterpretation, or errors in analysis.

Now let’s dive into the step-by-step reasoning of why we use schema validation:

Defining the Schema: The first step is to define what a valid dataset should look like. For example, if you have a customer table, your schema might specify that it must include fields like customer_id, name, email, and phone_number. Each field has specific data types (e.g., integer for ID, string for text).

Ingesting the Data: As the data is ingested from various sources—like CSV files, databases, APIs, or streaming systems—the schema validation process kicks in to ensure that each piece of data conforms to these rules.

Validation Process: During this process, each record is checked against the schema:

Field Existence and Type Matching: Ensure all required fields are present and their types match what’s expected.
Data Constraints: Check if values adhere to any additional constraints (e.g., email format validation).

Handling Invalid Data: If a piece of data fails validation, it can be handled in several ways:

Reject the Record: Automatically discard records that don’t meet the schema requirements.
Log and Alert: Log details about the invalid record for further investigation or alert relevant teams.
Transform and Save: Attempt to fix the issue (e.g., correcting a typo) before saving it.

By running schema validation checks, we can:

Ensure Data Consistency: Prevent inconsistencies that could arise from incorrect data formats.
Improve Data Quality: Enhance overall data quality by ensuring only well-formed data is used for analysis or storage.
Avoid Errors in Analysis: Reduce the risk of errors in downstream systems where data is used, such as in machine learning models or BI dashboards.

In summary, schema validation checks are crucial because they provide a systematic way to ensure that the data ingested into a system is structured and formatted correctly. This not only maintains data quality but also prevents potential issues that could arise from malformed or incorrectly formatted data.

#### Common mistakes
often include:

Failing to define a clear schema upfront.
Not implementing real-time validation checks during ingestion.
Overlooking edge cases in the validation rules, leading to incomplete handling of invalid data.

