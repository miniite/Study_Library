---
type: "practice"
difficulty: "Intermediate"
---

# How would you split a vague ticket like 'make the dashboard faster' into concrete prompts for an AI?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Prompting for Trade-off Analysis](<../../../../09_Software_Engineering/Developer_Productivity/AI_Assisted_Coding/Prompting_for_Trade-off_Analysis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To tackle the vague task of "make the dashboard faster," we need to break it down into more specific and actionable prompts that can guide the AI assistant, such as Copilot or similar tools. The goal is not only to improve performance but also to ensure a user-friendly experience. Here’s how you might approach this from first principles:

Understand the Scope: Determine what part of the dashboard needs optimization—could be loading time, responsiveness on interactions, or specific queries.
Identify Performance Metrics: Define key metrics like load times, response times, and query execution speeds to measure progress.
Prioritize Features: Focus on critical features that impact user experience most significantly.
Provide Context: Include relevant context such as the database schema, API usage patterns, and any known performance bottlenecks.

**Comparison:**

```
Prompt	Description
Optimize SQL Queries	Analyze and refactor SQL queries used in fetching data from the 'dashboard' table.
Cache API Responses	Identify and cache frequently accessed API endpoints to reduce latency.
Lazy Loading Strategy	Introduce lazy loading for non-critical components of the dashboard to improve initial load time.
```

**Code:**

```python
# Example code snippet to fetch data from a hypothetical dashboard query
def get_dashboard_data():
    # Fetch data from database or API
    response = db.query("SELECT * FROM dashboard WHERE ...")
    return response
```

**Example input / output:**

```
Input: A vague ticket like "make the dashboard faster."
Output:
CODE
Copy
[Prompt1]: Optimize the query for fetching user page views from the 'dashboard' table to improve load times.
[Prompt2]: Reduce latency in API calls by caching frequently accessed data.
[Prompt3]: Implement lazy loading for less critical sections of the dashboard to enhance overall responsiveness.
```

**Data:**

```yaml
database_schema:
  - name: "dashboard"
    columns: ["user_id", "page_views", "timestamp"]
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. No
6. Yes
7. No
8. Yes
9. Start
10. Is the task clear?
11. Break down the task intospecific prompts
12. Have performance metricsdefined?
13. Define key performancemetrics (e.g., load time,response times)
14. Are critical features identified?
15. Identify and prioritize criticaldashboard features impactinguser experience
16. Provide context to the AIassistant?
17. Include relevant context (e.g.,database schema, API usagepatterns)
18. End

