---
type: "practice"
difficulty: "Intermediate"
---

# How would you describe the Inmon approach and one way it differs from Kimball?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Kimball Approach](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Kimball_Approach.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let’s first summarize the key differences between Ralph Kimball's and W.H. Inmon's approaches to data modeling, focusing on how they differ in their foundational concepts of star schema and snowflake schema.

Inmon is known for his comprehensive approach where a single Data Warehouse (DW) contains all the necessary data from multiple operational systems. According to Inmon, data should be stored in a normalized way initially before denormalizing it into a star schema. The star schema consists of one fact table at the center surrounded by dimension tables, each representing different attributes about the business entities.

In contrast, Kimball's approach emphasizes building a series of smaller, more focused data marts from which users can derive insights. His methodology starts with a snowflake schema, where the dimensions are normalized and can be further divided into hierarchies. Snowflake schemas tend to have multiple levels of dimension tables, forming a kind of “snowflaking” effect.

The core difference lies in their starting points: Inmon begins with normalization and then denormalizes for performance, while Kimball starts by normalizing the dimensions themselves, creating a more hierarchical structure.

**Comparison:**

```
FactTable Star Schema	FactTable Snowflake Schema
FactKey, Dimension1Key, Dimension2Key	FactKey, DimProductKey, DimDateKey
	DimProduct (BrandKey, CategoryKey)
	DimBrand
	DimCategory
	DimDate (MonthKey, YearKey)
	DimMonth

This table illustrates the structure difference between star and snowflake schemas.
```

**Code:**

```text
# Example of Star Schema (Inmon Approach)
FactTable
- FactKey
- Dimension1Key
- Dimension2Key

Dimension1
- DimKey
- DimAttribute1
- DimAttribute2

Dimension2
- DimKey
- DimAttribute3
- DimAttribute4

# Example of Snowflake Schema (Kimball Approach)
FactTable
- FactKey
- DimProductKey
- DimDateKey

DimProduct
- ProductKey
- BrandKey
- CategoryKey

DimBrand
- BrandKey
- BrandName

DimCategory
- CategoryKey
- CategoryName

DimDate
- DateKey
- MonthKey
- YearKey

DimMonth
- MonthKey
- MonthName
```

