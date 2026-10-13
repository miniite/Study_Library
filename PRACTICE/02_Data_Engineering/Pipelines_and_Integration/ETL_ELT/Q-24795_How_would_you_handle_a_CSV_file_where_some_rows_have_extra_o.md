---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle a CSV file where some rows have extra or missing columns?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Source Changes and Spikes](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle a CSV file with inconsistent column counts, we need to understand the data structure and decide on a strategy that balances accuracy and performance. This involves checking for inconsistencies, handling missing values, and deciding whether to keep or drop extra columns. We'll use Python with Pandas for this example due to its robust data manipulation capabilities.

