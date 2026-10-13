---
type: "practice"
difficulty: "Deep"
---

# What is a chasm trap and a fan trap in dimensional models, and how have you avoided them in a real design?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Special Dimension Types](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Special_Dimension_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Chasm traps and fan traps are common pitfalls in data modeling within dimensional models, particularly star and snowflake schemas. These issues arise when designing a fact table that does not capture all possible combinations of dimensions, leading to incomplete or misleading data.

#### Triage

The question is probing your understanding of how to avoid incomplete data in dimensional models by ensuring that every combination of dimension values can be represented accurately. The interviewer might also be interested in your ability to design robust and comprehensive fact tables.

#### First-Principles Solution
1. Understanding Chasm Traps

A chasm trap occurs when a product or service offering is complex, leading to gaps in the data because not all customers use every possible combination of features. For example, if you have a subscription service with multiple optional add-ons, not everyone will select each one, causing holes in the fact table.

2. Understanding Fan Traps

A fan trap happens when there are many combinations of dimension values that should be represented but are missed due to poor design choices. This can occur when too much normalization is applied or dimensions are split incorrectly. For instance, in a customer purchase history fact table, missing combinations of product categories and regions could lead to an incomplete picture.

Avoiding Chasm Traps

Identify Key Product Features: Begin by understanding the core features of your products. If there's a complex feature that customers might not use universally, consider how you can still capture data for all users.

Use Bridge Tables: For chasm traps involving optional features, create bridge tables to ensure all possible combinations are represented even if not every user selects them.

Regular Audits and Analytics: Continuously review the data to identify any gaps that might be indicative of chasm traps. Use business rules and analytics to detect incomplete data patterns.

Avoiding Fan Traps

Comprehensive Dimension Modeling: Ensure that all dimensions are properly normalized but not over-normalized. Balance between normalization and denormalization based on the number of combinations and their business impact.

Dimension Splitting Carefully: Only split dimensions when necessary to avoid creating too many detailed levels. Over-splitting can lead to fan traps.

Use Star Schema with Snowflake Structure: A snowflake schema can help manage complexity by splitting some dimension tables, but ensure that the core fact table remains simple and comprehensive.

#### Example Scenario

In a retail company tracking customer purchases, we noticed missing combinations of product categories and regions over time. By reviewing the data, we identified several instances where customers did not purchase products from certain regions or categories, leading to incomplete sales data. We addressed this by implementing a bridge table that captured all possible category-region combinations, even if no purchases occurred.

#### Closing Summary

To avoid chasm traps and fan traps, it’s essential to understand the complexity of your product offerings and design fact tables that can handle all possible combinations. Regular audits and careful dimension modeling are crucial in ensuring comprehensive data representation.

#### Common Mistakes/Follow-Ups
Over-Normalization: Ensuring dimensions aren't split too finely.
Incomplete Data Audits: Failing to regularly review the data for gaps.
Ignoring Business Rules: Not incorporating business knowledge into data model design.

**Comparison:**

```
Product	Categories	Regions
TV	Electronics	North America
Shirt	Clothing	Europe
Purchases
[ProductID, CustomerID, Quantity]
(1, 101, 1) for a customer in NA purchasing a TV
```

**Code:**

```sql
-- Example of a bridge table
CREATE TABLE ProductCategoryRegion (
    ProductID INT,
    CategoryID INT,
    RegionID INT,
    PRIMARY KEY (ProductID, CategoryID, RegionID)
);
```

**Example input / output:**

```text
Input:
- Customer A purchased an electronic device from North America.
- Customer B did not make any purchase in Europe.

Output:
FactTable:
- [CustomerAId, ProductCategoryRegionId, Quantity] => (101, 1, 1)

BridgeTable:
- [ProductID, CategoryID, RegionID] => (1, 1, 101) for an electronic device in North America
```

**Data:**

```json
{
  "categories": [
    {"id": 1, "name": "Electronics"},
    {"id": 2, "name": "Clothing"},
    {"id": 3, "name": "Books"}
  ],
  "regions": [
    {"id": 101, "name": "North America"},
    {"id": 102, "name": "Europe"},
    {"id": 103, "name": "Asia"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. FactTable
4. Chasm Trap?
5. BridgeTable
6. Comprehensive Design
7. Ensure All CombinationsCovered
8. Regular Audits

