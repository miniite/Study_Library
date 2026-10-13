---
type: "practice"
difficulty: "Intermediate"
---

# How do you safely convert a badly formatted date string into a proper DATE type in SQL when the source has mixed formats?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Cleaning Text and Numeric Fields](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To safely convert a badly formatted date string into a proper DATE type in SQL, especially when dealing with mixed formats, we need to employ a multi-step approach that involves preprocessing and pattern matching. The core idea is to first identify the format of each date string, then apply the appropriate conversion function, and finally ensure no data loss or corruption occurs during the process.

Let's assume you have a table events with a column event_date containing mixed date formats like '2023-10-05', 'Oct 5th, 2023', '5/Oct/2023', etc. Our goal is to convert these into the DATE format.

#### Triage

The question asks how to handle a column with mixed date formats in SQL and safely convert it into a proper DATE type. The interviewer might be probing your understanding of SQL functions, date formatting patterns, and error handling techniques.

#### First-Principles Solution
Identify the Formats: Use regular expressions or pattern matching to identify different date formats.
Convert Each Format: Apply the appropriate conversion function for each format.
Validate Conversions: Ensure all conversions are successful; handle any failed conversions appropriately.
Store Clean Data: Store the cleaned data in a new column or overwrite the original one.
Step-by-Step Solution

Identify Formats Using Regular Expressions:

Format 1: YYYY-MM-DD
Format 2: Month DD, YYYY (e.g., 'Oct 5th, 2023')
Format 3: DD/MM/YYYY (e.g., '5/Oct/2023')

Convert Each Format:

SQL
-- Step 1: Split the date string to extract month, day, and year if needed.
WITH parsed_dates AS (
    SELECT
        event_date,
        CASE
            WHEN event_date ~* '^[0-9]{4}-[0-9]{2}-[0-9]{2}$' THEN
                TO_DATE(event_date, 'YYYY-MM-DD')
            WHEN event_date ~* '^[A-Za-z]+ [0-9]+, [0-9]{4}' THEN
                TO_DATE(event_date, 'Month DD, YYYY')
            WHEN event_date ~* '^[0-9]+/[A-Za-z]+/[0-9]{4}' THEN
                TO_DATE(SUBSTR(event_date, 1, 2) || '/' || SUBSTR(event_date, -4) || '/' || SUBSTR(event_date, 7, 2), 'DD/MM/YYYY')
            ELSE NULL
        END AS clean_date
    FROM events
)
-- Step 2: Filter out rows where conversion failed.
SELECT
    event_date,
    clean_date
FROM parsed_dates
WHERE clean_date IS NOT NULL;

Handle Failed Conversions:

You can log the failed conversions or store them in a separate column for further analysis.

Store Clean Data:

SQL
-- Insert cleaned data into a new column.
ALTER TABLE events ADD COLUMN clean_event_date DATE;

UPDATE events SET
    clean_event_date = (
        SELECT clean_date FROM parsed_dates WHERE parsed_dates.event_date = events.event_date AND clean_date IS NOT NULL
    );

#### Closing Summary
and Common Mistakes/Follow-Ups

This approach ensures that each date string is handled appropriately based on its format. However, it's important to consider edge cases like invalid dates (e.g., '13/02/2023') which might not match any pattern and should be treated as errors.

#### Common mistakes
include:

Overlooking edge cases.
Not handling null values properly.
Failing to validate the converted date before storing it.

#### Common Mistakes/Follow-Ups
Edge Cases: Ensure you handle dates that don't fit into any of the given formats (e.g., '02/13/2023' which is not a valid month).

#### Performance Considerations
If dealing with large datasets, consider using partitioning or indexing.
Error Handling: Implement logging for failed conversions to debug and fix issues.

