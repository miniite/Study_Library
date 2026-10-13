---
type: "practice"
difficulty: "Intermediate"
---

# How would you monitor row counts over time to catch a day where far fewer rows arrived than usual?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Detecting Unit Changes](<../../../../02_Data_Engineering/Data_Quality/Validation/Detecting_Unit_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure data integrity and identify anomalies in our data pipelines, monitoring row counts is crucial. This process involves setting up automated checks that compare current row counts against historical patterns to detect any significant deviations. In this solution, we'll monitor the number of rows arriving on a daily basis by leveraging a simple yet effective approach.

#### Triaging the Question

The core of the question is about identifying irregularities in data arrival rates, which could indicate issues like data ingestion problems or unexpected system downtimes. The key steps involve:

Defining Normal Behavior: Establishing what constitutes normal row count behavior.
Monitoring Mechanism: Setting up a mechanism to track and compare current counts against historical data.
Thresholds & Alerts: Defining thresholds that trigger alerts when rows are significantly lower than expected.

#### First-Principles Solution
Step 1: Define Normal Behavior

To determine what's "normal," we need historical data on typical row counts. Let’s assume our pipeline processes a steady number of rows daily, and any deviation more than 20% from the average could be considered abnormal.

Step 2: Monitoring Mechanism

We can use a database system like PostgreSQL or a time-series database to store and query this data. We'll create a table that logs daily row counts and their timestamps.

SQL
CREATE TABLE daily_row_counts (
    day DATE PRIMARY KEY,
    count BIGINT
);

Each day, after the pipeline runs, we insert a new entry with the current date and the total number of rows processed:

SQL
INSERT INTO daily_row_counts (day, count) VALUES ('2023-10-15', 10000);
Step 3: Thresholds & Alerts

We’ll set up an alert system that triggers when the row count is significantly lower than expected. For instance, if the average daily count for a particular day of the week is usually around 9500 rows and today’s count is only 4750 (25% lower), we need to be notified.

To implement this, we can create a function that checks the current row count against historical data:

SQL
CREATE OR REPLACE FUNCTION check_row_count() RETURNS void AS $$
BEGIN
    -- Assume we have a function to get average daily count for today's day of week
    IF (SELECT count FROM daily_row_counts WHERE day = CURRENT_DATE - INTERVAL '1 day') * 0.8 <
       (SELECT count FROM daily_row_counts WHERE day = CURRENT_DATE) THEN
        RAISE NOTICE 'Row count anomaly detected: % vs. expected %',
                      (SELECT count FROM daily_row_counts WHERE day = CURRENT_DATE),
                      (SELECT count FROM daily_row_counts WHERE day = CURRENT_DATE - INTERVAL '1 day') * 0.8;
    END IF;
END $$ LANGUAGE plpgsql;

This function compares today’s row count with the previous day's count adjusted by a factor, triggering an alert if the current count is too low.

Step 4: Integrate into Pipeline

We'll integrate this check into our ETL pipeline so that it runs at the end of each batch process. This ensures we get real-time notifications and can take corrective actions as soon as possible.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, monitoring row counts involves setting up a historical baseline for what’s normal, defining thresholds to detect anomalies, and implementing automated alerts to notify us when something is off. Common mistakes include not having a robust system for storing historical data or ignoring small but significant deviations.

During the interview, you might be asked about:

How to handle edge cases like weekends or holidays where row counts naturally vary.
What tools can be used instead of SQL databases (e.g., AWS CloudWatch).
If we should consider more complex statistical models beyond simple thresholds.
Strategies for mitigating false positives and negatives.

