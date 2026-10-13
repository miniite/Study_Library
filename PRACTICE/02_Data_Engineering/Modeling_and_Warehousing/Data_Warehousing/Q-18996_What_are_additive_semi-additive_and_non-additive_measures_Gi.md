---
type: "practice"
difficulty: "Deep"
---

# What are additive, semi-additive, and non-additive measures? Give an example of each.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Dimension Integrity](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/Fact_Dimension_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In data warehousing, measures (or metrics) can be classified into three categories based on their behavior when aggregated over different time periods or dimensions: additive, semi-additive, and non-additive measures. Understanding these distinctions is crucial for designing effective data warehouses that provide accurate insights.

#### Triaging the Question

The question asks you to explain three types of measures in a data warehouse: additive, semi-additive, and non-additive. The interviewer likely wants to assess your understanding of how different metrics behave when aggregated over varying dimensions. This knowledge is essential for designing an efficient and correct data model that can support various analytical queries.

#### First-Principles Solution
1. Additive Measures
Definition: An additive measure is one where the sum of the values over a period or across different dimensions provides a meaningful result.

#### Example
Sales revenue, total cost, and number of transactions are all additive measures because you can add them up without losing accuracy.
2. Semi-Additive Measures
Definition: A semi-additive measure is one where the sum of values over a period is not meaningful, but the sum across different dimensions is.

#### Example
Inventory levels or stock count are typically semi-additive because adding inventory counts from different locations doesn't give you the total inventory at any given point in time.
3. Non-Additive Measures
Definition: A non-additive measure cannot be summed up meaningfully, neither over a period nor across dimensions.

#### Example
Market share or customer satisfaction scores are non-additive because adding them does not make sense; the result is meaningless and could be misleading.

#### Closing Summary

Understanding these types of measures is critical for data engineers to design accurate and efficient data warehousing solutions. By correctly classifying metrics, you ensure that queries produce meaningful results without leading to errors or misinterpretations.

#### Common Mistakes/Follow-Ups
Misclassifying Measures: Sometimes, developers confuse semi-additive with additive measures.
Ignoring Non-Additivity: Failing to recognize non-additive measures can lead to incorrect aggregation and analysis.
Complexity in Design: Understanding these distinctions helps in designing more complex data models that support various types of queries.

#### Example Scenario
s
Additive Measure Example:
PLAINTEXT
Total Sales Revenue: $1,000 (Q1) + $2,000 (Q2) = $3,000 (Annual)
Semi-Additive Measure Example:
PLAINTEXT
Total Inventory: 500 Units (Location A) + 700 Units (Location B) ≠ 1,200 Units (Total Location)
Non-Additive Measure Example:
PLAINTEXT
Customer Satisfaction Score: 80% (Q1) + 90% (Q2) ≠ 170% (Annual)
Self-Check

This answer covers all the key points and provides clear examples to illustrate each type of measure.

