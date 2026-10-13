---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git log with a filter to find every commit where you changed an ingestion or loading script over the last month?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to understand how git log works and its filtering capabilities, particularly within the context of a specific time range. Essentially, you want to find all commits that modified scripts responsible for data ingestion or loading over the last month.

Firstly, let's break down what needs to be done:

Identify the file patterns associated with ingestion/loading scripts.
Use git log to filter by these file patterns and a specified time range.

We'll start by identifying common naming conventions or directories that might contain such scripts. For instance, they could be named as ingest, load, or located in a specific directory like /scripts/ingestion.

Next, we will use the command line to construct our git log query with appropriate filters. Here’s how you can do it:

