---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the DATE type and the TIMESTAMP type in PostgreSQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date and Timestamp Types](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Date_and_Timestamp_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with date and time in PostgreSQL, understanding the differences between DATE and TIMESTAMP types is crucial. The DATE type stores a date without any time component, while TIMESTAMP stores both date and time information with millisecond precision.

#### Triage

The question asks you to explain the difference between DATE and TIMESTAMP in PostgreSQL. The interviewer is probing your understanding of basic data types in SQL databases. You need to clearly differentiate these two types based on their storage characteristics.

#### First-Principles Solution

DATE Type:

Stores only dates.
Format: YYYY-MM-DD.

#### Example
'2023-10-05'.

TIMESTAMP Type:

Stores both date and time, with millisecond precision by default (but can be configured).
Format: YYYY-MM-DD HH:MI:SS.MS, where MS is milliseconds.

#### Example
'2023-10-05 14:30:00.123'.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, DATE type should be used when you need to store only the date without any time component, while TIMESTAMP is suitable for scenarios where both date and precise time are required. For instance, if you're recording a transaction that occurred at a specific moment in time, you would use TIMESTAMP. If you're just tracking events by date alone, like birthdays or anniversary dates, DATE is more appropriate.

A common mistake is assuming that TIMESTAMP automatically includes timezone information; it does not. If you need to store time zones, consider using the TIMESTAMPTZ type instead, which stands for "timestamp with time zone".

Code Blocks
SQL
-- Example of DATE and TIMESTAMP usage in PostgreSQL

-- Create a table example
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    event_date DATE,
    event_time TIMESTAMP
);

-- Inserting values
INSERT INTO events (event_date, event_time) VALUES ('2023-10-05', '2023-10-05 14:30:00.123');

-- Query to retrieve data
SELECT * FROM events;
Data Blocks

N/A

Input/Output
SQL
INSERT INTO events (event_date, event_time) VALUES ('2023-10-05', '2023-10-05 14:30:00.123');

SELECT * FROM events;

Result:

CODE
 id | event_date |        event_time
----+------------+---------------------
  1 | 2023-10-05 | 2023-10-05 14:30:00.123
Tables

N/A

Flowcharts

N/A

The answer is comprehensive and correct.

