---
type: "practice"
difficulty: "Expert"
---

# With CDC-driven SCD, out-of-order events can close a version before its update arrives — how do you handle ordering and watermarking?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In slowly changing dimension (SCD) scenarios with change data capture (CDC), maintaining temporal consistency is crucial, especially when out-of-order events can close a version before its update arrives. This situation arises in various real-world applications where events are captured at different times and processed asynchronously. To handle such ordering issues and ensure consistent state management, we need to implement robust watermarking mechanisms.

Firstly, let's clarify what we're dealing with:

Problem: Out-of-order CDC events that close a version before their corresponding update arrives.
Objective: Ensure temporal consistency by managing version lifetimes effectively.
Key Concept: Watermarking, which is used to track the latest known state of data.

To handle this problem, we need to implement two main strategies:

Watermark Management: Track the last known valid timestamp for each dimension value.
Event Ordering and Processing: Ensure events are processed in chronological order, even if they arrive out-of-order.

Now, let's break down the solution from first principles:

Step-by-Step Solution
1. Define Watermarks

Watermarks serve as a mechanism to track the latest known state of data. For each dimension value, we maintain a watermark that represents the last valid time when the record was updated or closed.

SQL
CREATE TABLE Customer (
    customer_id INT,
    name STRING,
    version INT,
    watermark TIMESTAMP
);
2. Event Processing and Watermark Update

When an event arrives, it updates the state of the dimension table. However, if a closing event (indicating that a version is no longer valid) arrives before its corresponding update event, we must ensure that our processing logic handles this correctly.

SQL
-- Example: Handling a closing event
UPDATE Customer SET version = 2, watermark = '2023-10-01T14:00' WHERE customer_id = 1;

-- Example: Handling an updating event
UPDATE Customer SET name = 'New Name', version = 3 WHERE customer_id = 1 AND version < 2;
3. Ensuring Event Ordering and Processing

To ensure that events are processed in the correct chronological order, we can use a watermark-based approach combined with a processing time or event-time timestamp.

SQL
-- Example: Using a WATERMARK clause to enforce temporal constraints
INSERT INTO Customer (customer_id, name, version, watermark)
SELECT customer_id, name, version, watermark
FROM RawEvents
WHERE event_time >= lag(watermark) OVER (PARTITION BY customer_id ORDER BY event_time);

#### Closing Summary
and Common Mistakes

To summarize, handling out-of-order CDC events in SCD requires a robust watermarking mechanism to track the latest valid state of each dimension. This ensures that even if closing events arrive before their corresponding updates, we can still maintain consistent temporal data.

#### Common mistakes
include:

Ignoring Watermarks: Not maintaining and updating watermarks correctly.
Inconsistent Processing Order: Failing to ensure that events are processed in the correct chronological order.
Lack of Timestamps: Not using timestamps effectively to enforce temporal constraints.

By implementing these strategies, we can handle out-of-order CDC events gracefully and maintain a consistent state in our SCD dimensions.

