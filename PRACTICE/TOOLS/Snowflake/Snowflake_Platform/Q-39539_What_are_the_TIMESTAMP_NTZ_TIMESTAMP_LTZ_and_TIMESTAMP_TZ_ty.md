---
type: "practice"
difficulty: "Intermediate"
---

# What are the TIMESTAMP_NTZ, TIMESTAMP_LTZ, and TIMESTAMP_TZ types and when do you use each?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Time Travel and Storage Costs](<../../../../TOOLS/Snowflake/Snowflake_Platform/Time_Travel_and_Storage_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In Snowflake, TIMESTAMP_NTZ, TIMESTAMP_LTZ, and TIMESTAMP_TZ are used to store date-time values but with different time zones handling mechanisms. TIMESTAMP_NTZ stores date-time in UTC without any timezone information, TIMESTAMP_LTZ stores the date-time with a local timestamp (usually UTC) and includes microsecond precision, while TIMESTAMP_TZ stores date-time with explicit timezone information.

#### Triaging the Question
The question is asking about three different timestamp types (TIMESTAMP_NTZ, TIMESTAMP_LTZ, and TIMESTAMP_TZ) in Snowflake. The interviewer might be probing your understanding of how these types work, their differences, when to use them, and potential implications on data processing.

Key Assumptions:

Context: This is relevant for a Data Engineer role.
Level: Intermediate experience means you should understand the core concepts but also know practical scenarios where each type would be used.

Step-by-Step Solution:

1. TIMESTAMP_NTZ (Non-Time Zone)
Definition: Stores date-time values in UTC without any time zone information.
Use Case: Ideal for raw data storage, particularly when dealing with global datasets that don’t have a specific time zone context or where the application will handle timezone conversion.

#### Example
SQL
CREATE TABLE users (
    id INT,
    created_at TIMESTAMP_NTZ
);
2. TIMESTAMP_LTZ (Local Time Zone)
Definition: Stores date-time values with a local time zone, usually UTC, and includes microsecond precision.
Use Case: Common for storing timestamps in applications that operate on the assumption of UTC but need to display data in user-local time zones.

#### Example
SQL
CREATE TABLE transactions (
    id INT,
    transaction_time TIMESTAMP_LTZ
);
3. TIMESTAMP_TZ (Time Zone Aware)
Definition: Stores date-time values with explicit timezone information, allowing for precise conversion between time zones.
Use Case: Used when the application needs to store and process data with specific time zone contexts, such as financial or regulatory requirements where exact time zone is critical.

#### Example
SQL
CREATE TABLE flights (
    id INT,
    departure_time TIMESTAMP_TZ
);
Practical Scenarios:
Use TIMESTAMP_NTZ when you want to store date-time in a single, unambiguous format (UTC) and don’t need time zone conversions.
Use TIMESTAMP_LTZ for applications that operate on the assumption of UTC but require user-local display without explicit timezone awareness.
Use TIMESTAMP_TZ for scenarios where you must store data with specific time zones and ensure precise timezone handling.

#### Common Mistakes
and Follow-Ups:
Mistake: Confusing TIMESTAMP_LTZ and TIMESTAMP_TZ. Both include timezone information, but TIMESTAMP_LTZ is usually stored as UTC.

#### Follow-Up
How do you convert between these types in Snowflake? For instance, converting from TIMESTAMP_NTZ to TIMESTAMP_LTZ.
Advanced Follow-Up: Discuss the implications of using different time zones on data processing and storage costs.

