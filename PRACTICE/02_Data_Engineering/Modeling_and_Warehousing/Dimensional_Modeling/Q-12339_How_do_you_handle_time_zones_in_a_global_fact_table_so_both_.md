---
type: "practice"
difficulty: "Deep"
---

# How do you handle time zones in a global fact table so both local-time and UTC reporting are correct, and where do you store the offset?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Transaction vs Snapshot Facts](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Transaction_vs_Snapshot_Facts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a global fact table that needs to support both local-time and UTC reporting, handling time zones correctly is crucial. The key challenge here is ensuring that the data can be accurately converted between different time zones while maintaining the integrity of the timestamps in their original form. This involves understanding how time zones are represented and stored, as well as the design choices for storing offsets and managing conversions.

In this scenario, we're dealing with two primary reporting needs: local-time (the time zone specific to each region where data is entered) and UTC (Coordinated Universal Time, which is used for international standardization). The goal is to store the time in a format that can be easily converted between these representations without losing accuracy.

#### Triage

The question essentially asks how to design a schema and logic to handle time zones effectively. We need to ensure:

Data Storage: How do we store timestamps such that they are consistent across different time zones?
Time Zone Offsets: Where do we store the offset information for each data entry?
Conversion Logic: How do we convert between local-time and UTC?

#### First-Principles Solution
1. Data Storage

To handle both local-time and UTC, we should store timestamps in a way that is agnostic to time zones but can be converted as needed. The most straightforward approach is to store the timestamp along with its corresponding time zone ID or offset.

For storage, consider using two fields:

timestamp: Stores the actual date and time of the event.
timezone_id or offset_hours_minutes: Stores the time zone identifier (e.g., America/New_York) or the offset in hours and minutes from UTC.
2. Time Zone Offsets

Time zones can be stored as either an identifier or a fixed offset. However, using identifiers is generally more robust because offsets can change over time due to daylight saving time adjustments. For example:

Using Identifiers: Store timezone_id (like 'America/New_York') in the database.
Using Offsets: Store offset_hours_minutes (like -05:00 for Eastern Standard Time).
3. Conversion Logic

To convert between local-time and UTC, we need to handle time zone conversions dynamically. This can be achieved by:

Storing Local-Time Data with Offset/Identifier: When data is entered, store the timestamp along with the timezone_id or offset_hours_minutes.
Converting Between Time Zones:
To get local-time from UTC: Use the offset_hours_minutes to adjust the UTC time to the local time zone.
To get UTC from local-time: Use the reverse process by subtracting the offset_hours_minutes.
Detailed Implementation

Let’s consider a simplified example schema:

SQL
CREATE TABLE fact_table (
    id INT PRIMARY KEY,
    timestamp TIMESTAMP,  -- Stores the actual date and time of the event
    timezone_id VARCHAR(50),  -- Time zone identifier (e.g., 'America/New_York')
    local_time TIMESTAMP,  -- Optional: Store the local-time if needed for quick queries
    utc_time TIMESTAMP GENERATED ALWAYS AS (timestamp AT TIME ZONE timezone_id) STORED  -- UTC time derived from timestamp and timezone_id
);

In this schema:

timestamp is stored as the actual event date and time.
timezone_id stores the time zone identifier.
local_time can be optionally used for quick local-time queries if required.
utc_time is generated using SQL’s AT TIME ZONE function, which converts the timestamp to UTC based on the timezone_id.

#### Common Mistakes
and Follow-Ups
Ignoring Time Zone Changes: Ensure that your application handles changes in time zones due to daylight saving time adjustments or legislative changes.
Incorrect Offset Storage: Storing offsets instead of identifiers can lead to inaccuracies if the offset changes over time.

#### Performance Considerations
Storing local_time along with timestamp and timezone_id can improve query performance but requires careful management of space and updates.

In summary, handling time zones in a global fact table involves storing both timestamps and timezone identifiers or offsets to ensure accurate conversions between local-time and UTC.

**Code:**

```sql
CREATE TABLE fact_table (
    id INT PRIMARY KEY,
    timestamp TIMESTAMP,
    timezone_id VARCHAR(50),
    local_time TIMESTAMP,
    utc_time TIMESTAMP GENERATED ALWAYS AS (timestamp AT TIME ZONE timezone_id) STORED
);
```

