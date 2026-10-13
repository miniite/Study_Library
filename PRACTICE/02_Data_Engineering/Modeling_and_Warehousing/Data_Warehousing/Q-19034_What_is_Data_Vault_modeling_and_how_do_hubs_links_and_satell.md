---
type: "practice"
difficulty: "Deep"
---

# What is Data Vault modeling and how do hubs, links, and satellites differ from the star schema?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Distribution Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Distribution_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Data Vault modeling is a highly flexible data warehousing technique that emphasizes physical decoupling of logical structures to ensure robustness, scalability, and ease of maintenance. Hubs, links, and satellites are key components in this model, each serving distinct roles compared to the more rigid star schema.

#### Triaging the Question

The question asks you to explain Data Vault modeling and compare it with the star schema using hubs, links, and satellites. The core aspects we'll cover include:

Understanding Star Schema: A traditional dimensional modeling approach.
Introduction to Data Vault Modeling: Its principles and benefits.
Key Components in Data Vault:
Hubs: Central entities that store unique identifiers for all related entities.
Links: Connect hubs with each other, representing many-to-many relationships.
Satellites: Store attributes (dimensions) associated with hubs or links.

#### First Principles Solution
Star Schema Overview

A star schema is a common dimensional data model where facts are aggregated around dimensions. Each fact table (central "star") has multiple dimension tables, and all relate to the same grain of granularity. This structure is straightforward but can become inflexible as the number of dimensions grows.

Data Vault Modeling: A Flexible Approach

Data Vault modeling introduces flexibility by decoupling logical structures from physical schemas:

Hubs: Store unique identifiers for each entity.
Links: Connect hubs, representing many-to-many relationships without embedding details in the model.
Satellites: Hold detailed attributes (dimensions) related to hubs or links.
Hubs

Hubs are central entities that store surrogate keys (unique identifiers). This ensures a consistent primary key across the entire data warehouse:

Primary Key
Hub
Surrogate Key

#### Example
An Orders hub might contain only order IDs, ensuring uniqueness and consistency.

Links

Links represent many-to-many relationships between hubs. They do not include attributes but merely connect entities:

Many-Many
Many-Many
Hub1
Link
Hub2

#### Example
An Order_Hub_OrderLineItem link connects order IDs (from the Orders hub) with line item IDs.

Satellites

Satellites store detailed attributes related to hubs or links. They ensure that all business dimensions are stored in one place:

Many-Many
Many-Many
Hub
Link
Hub

#### Example
A Order_Satellite might store order quantities, prices, etc., associated with each hub.

#### Closing Summary

Data Vault modeling provides a flexible and robust approach to data warehousing by decoupling logical structures from physical schemas. Hubs, links, and satellites ensure that the model is easy to maintain and scale while preserving the integrity of unique identifiers and business attributes.

#### Common Mistakes
and Follow-Ups
Ignoring Business Attributes: Forgetting to store detailed attributes in satellites can lead to data loss.
Over-Complexity with Links: Overusing links instead of embedding details can complicate queries.
Not Using Hubs for Unique Identifiers: Relying on natural keys rather than surrogate keys can introduce redundancy and inconsistency.

Always ensure that your model balances flexibility with maintainability, considering the specific needs of your data ecosystem.

