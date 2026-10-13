---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a query to find gaps in a sequence of order IDs?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Basic Query Practice](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Basic_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find gaps in a sequence of order IDs using PostgreSQL, we need to identify which order IDs are missing from a continuous series. This involves generating a list of expected order IDs and then comparing it with the actual data to spot any discrepancies.

The key idea here is to leverage window functions and common table expressions (CTEs) to create a list of all possible sequential numbers within a given range, compare them with the existing order IDs, and flag any missing values.

Let's assume we have an orders table with a column order_id. The steps are as follows:

Identify the minimum and maximum order ID in the dataset.
Generate a sequence of all possible order IDs within this range.
Use window functions to compare the generated sequence with the actual order IDs.

