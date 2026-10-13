---
type: "practice"
difficulty: "Basic"
---

# What is the difference between an inner and left join when enriching data during cleaning, and how does the choice affect row loss?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with datasets in data preprocessing and cleaning, you often need to merge or combine data from multiple sources to enrich it. Two common types of joins used in such scenarios are inner join and left join. The main difference between them lies in how they handle missing values and the rows that get included in the final dataset.

TL;DR
Inner Join: Only includes rows where there is a match in both datasets.
Left Join: Includes all rows from the left (first) dataset, even if no matching row exists in the right dataset. Rows without matches are filled with NULL values in the result.

#### Triaging the Question

The question is essentially asking you to differentiate between two types of SQL joins: inner and left join, focusing on their behavior during data enrichment tasks and how they impact the number of rows in the final dataset (i.e., row loss).

#### Assumptions
You are working with tabular data.
The datasets being joined have columns with common keys.

#### Solution

Let's dive into each type of join:

Inner Join:

This join returns only the rows that have matching values in both tables based on a specific key or column.
If there is no match, the row from one table will not be included in the result set.
For example, if you are enriching customer data with order history and using a customer ID as the key, an inner join would only return records where a corresponding order exists.

Left Join:

This join returns all rows from the left (first) dataset and the matching rows from the right dataset.
If there is no match in the right dataset, the result will still include all rows from the left dataset with NULL values for columns from the right dataset where no match exists.
For example, if you are enriching customer data with order history using a customer ID as the key and some customers have no orders, a left join would include these customers in the final dataset but mark their orders as NULL.
Impact on Row Loss
Inner Join: This can lead to significant row loss because only matching rows are retained. If many records in one table do not match any records in the other table, the resulting dataset will be smaller.
Left Join: This minimizes row loss by including all records from the left table. Even if there is no corresponding data in the right table, these records still appear in the result with NULL values.

#### Closing Summary

In summary, when enriching data during cleaning, choosing between an inner join and a left join depends on your specific needs:

Use inner join if you need to ensure that only fully matched records are included.
Use left join if you want to include all records from one dataset and fill in with NULL values where there is no match.

#### Common mistakes
often involve not considering the implications of row loss or assuming both types of joins behave similarly. Always review your data schema and cleaning requirements carefully before choosing a join type.

#### Common Mistakes/Follow-Ups
Mistake: Assuming that an inner join will always yield more accurate results without understanding potential data loss.

#### Follow-Up
Discuss edge cases where row loss might be acceptable (e.g., in preprocessing steps) versus situations where retaining all rows is crucial (e.g., when maintaining the integrity of your master dataset).

