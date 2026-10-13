---
type: "practice"
difficulty: "Deep"
---

# What is a factless fact table and give an example of when you would use one?

**Difficulty:** Deep · **Levels:** Fresher · [↑ Late Arriving Dimensions](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A factless fact table, also known as a skeleton fact table, is a type of fact table in a data warehouse that contains no actual facts (or measures). Instead, it serves to establish relationships between dimensions and other fact tables through keys. Factless fact tables are used when there's a need for a logical grouping or association among dimension records without any quantitative metrics.

#### Triage
What the Question is Asking

The question is probing your understanding of fact table concepts in data warehousing, specifically about scenarios where a factless fact table would be appropriate. The interviewer wants to assess your knowledge of how different types of tables (dimensions and facts) interact within a star or snowflake schema.

#### Assumptions
You understand the basic structure and purpose of fact tables.
You are familiar with dimension tables, which contain descriptive attributes.
You recognize that in data warehousing, fact tables typically store quantitative measures related to business metrics.
Step-by-Step Solution

Factless fact tables arise when you need a way to group multiple dimensions without any numerical values associated with them. Here’s an example of how and why they might be used:

Scenario: Customer Activity Tracking

Imagine a retail company that tracks various customer interactions across different channels (e.g., website, phone calls, in-store visits). Each interaction is logged as an event with relevant metadata like the date, time, type of channel, and additional descriptive fields about the customer.

However, there are some situations where you might want to create factless fact tables:

Cross-channel Analysis: You may need a central place to aggregate events from different channels without specific metrics.
Customer Journey Mapping: Understanding how customers move between different touchpoints (website -> phone call -> in-store) without needing numerical values for each transition.

#### Example
Schema

Let's consider the following schema:

SQL
CREATE TABLE CustomerInteractions (
    interaction_id INT PRIMARY KEY,  -- Unique identifier for each interaction
    customer_id INT,
    channel_type VARCHAR(50),        -- E.g., 'Website', 'Phone', 'In-Store'
    interaction_date TIMESTAMP       -- Date and time of the interaction
);

-- Factless fact table to track interactions across channels:
CREATE TABLE ChannelInteractions (
    interaction_id INT,             -- References CustomerInteractions.interaction_id
    store_id INT,
    phone_id INT,
    website_id INT,
    PRIMARY KEY (interaction_id)
);

In this example:

CustomerInteractions contains detailed records of customer interactions.
ChannelInteractions is a factless table that links the same interactions to different channels without any additional metrics.
Why Use a Factless Table?
Normalization: By separating the interaction record from the channel record, you normalize your data model.
Flexibility: You can easily add more dimensions or join with other tables as needed.
Scalability: Helps in managing large datasets by reducing redundancy.

#### Closing Summary

Factless fact tables are crucial for maintaining a flexible and scalable data warehouse schema. They allow you to establish relationships between different types of interactions without cluttering the model with unnecessary numerical values. This helps in creating more meaningful insights by focusing on how different dimensions relate to each other, rather than specific metrics.

#### Common Mistakes/Follow-Ups
Misunderstanding Dimensions and Facts: Always ensure that your understanding of dimension vs. fact tables is clear.
Overcomplicating the Schema: Sometimes, adding a factless table when it’s not necessary can make the schema overly complex.
Ignoring Business Logic: Ensure that any decision to use a factless table aligns with business requirements and data analysis needs.

