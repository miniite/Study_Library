---
type: "practice"
difficulty: "Basic"
---

# How do you remove duplicate rows in SQL using GROUP BY or DISTINCT, and when would you prefer one over the other?

**Difficulty:** Basic · **Levels:** Fresher · [↑ Reject vs Quarantine Bad Rows](<../../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Reject_vs_Quarantine_Bad_Rows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To remove duplicate rows in SQL, two common methods are GROUP BY with an aggregate function like COUNT and DISTINCT. Both achieve similar results but serve different use cases based on your specific requirements.

TL;DR:
GROUP BY: Useful for counting unique values or performing aggregations while keeping the original data.
DISTINCT: Simple to use for removing all duplicate rows, but doesn't support further processing with aggregate functions directly.

#### Triaging the Question

The question is essentially asking how to eliminate duplicates in a dataset and when to choose between GROUP BY and DISTINCT. The interviewer might be probing your understanding of SQL basic concepts, data integrity, and query optimization.

#### Assumptions
You have a table with some duplicate rows that you need to clean.
You want to understand the differences and trade-offs between using GROUP BY and DISTINCT.

#### First Principles Solution
Using DISTINCT

The simplest way to remove duplicates is by using the DISTINCT keyword. It returns all unique combinations of values from a single column or multiple columns.

SQL
SELECT DISTINCT column1, column2, column3 FROM your_table;

#### Example
Suppose you have a table named employees with potential duplicate entries:

id	name	department
1	Alice	HR
2	Bob	IT
3	Alice	HR
4	Carol	Marketing

Running the DISTINCT query would return:

id	name	department
1	Alice	HR
2	Bob	IT
3	Carol	Marketing

#### Pros

Easy to understand and implement.
No additional columns are returned.

#### Cons

Cannot include aggregate functions or other columns without SELECTing them explicitly.
May not be efficient for large datasets with many duplicates, as it needs to check each row individually.
Using GROUP BY

The GROUP BY clause can also remove duplicates by grouping rows based on a specific column or multiple columns and applying an aggregate function. This method is more flexible since you can perform additional operations like counting or summing up other columns.

SQL
SELECT column1, column2, COUNT(*) as count FROM your_table GROUP BY column1, column2;

#### Example
Using the same employees table:

Running this query would return a list of unique employees along with their counts:

name	department	count
Alice	HR	2
Bob	IT	1
Carol	Marketing	1

#### Pros

More powerful for complex data analysis.
Can include aggregate functions to get additional insights.

#### Cons

Requires a bit more complexity in writing the query.
May return unnecessary columns unless you are careful about what is selected.

#### Closing Summary
and Common Mistakes

To summarize, DISTINCT is simpler but less flexible, whereas GROUP BY with an aggregation function provides greater flexibility. The choice depends on whether you need to keep all original rows or perform further analysis.

#### Common Mistakes

Using GROUP BY without necessary aggregate functions.
Forgetting that DISTINCT only works on the columns specified in the query.
Not considering performance implications for large datasets.
Common Follow-Ups
How would you handle duplicates when there are more than two identical rows?
Can you provide an example where GROUP BY might be preferred over DISTINCT?

