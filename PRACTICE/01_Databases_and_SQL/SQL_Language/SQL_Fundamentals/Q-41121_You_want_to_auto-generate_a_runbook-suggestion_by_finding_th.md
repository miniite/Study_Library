---
type: "practice"
difficulty: "Deep"
---

# You want to auto-generate a runbook-suggestion by finding the most common remediation that resolved past incidents with the same signature. How would you write that SQL and rank candidate actions by historical success rate?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gaps and Islands](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Gaps_and_Islands.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify the most common remediations that resolved past incidents sharing the same signature. This involves querying a database of incident records and their resolutions, then analyzing the frequency and success rates of different actions taken.

First, let's break down what the question is asking:

We need to find incidents with the same "signature," which could be a combination of error codes, symptoms, or other identifiers.
For each signature, we want to identify the most common remediation that resolved the incident.
Finally, we rank these actions based on their historical success rate.

#### Assumptions

We have a table incidents with fields such as incident_id, signature, and resolution.
The resolution field contains the action taken to resolve the issue.
There is no explicit definition of what constitutes "success" for an incident, so we will assume that a remediation that was applied multiple times is more likely to be successful.

Now, let's reason through this step-by-step:

Identify Incidents by Signature: We need to group incidents by their signatures and count the frequency of each resolution.

Rank Resolutions by Frequency: For each signature, we want to find the top remediation that was applied most frequently.

Calculate Success Rate: To rank these resolutions by success rate, we would need a way to determine if an incident was resolved successfully. This could be based on follow-up logs or user feedback.

Here's how you might write this SQL query:

SQL
WITH signature_resolutions AS (
    SELECT
        signature,
        resolution,
        COUNT(*) AS frequency
    FROM incidents
    GROUP BY signature, resolution
),
resolution_successes AS (
    SELECT
        s.signature,
        r.resolution,
        COUNT(DISTINCT CASE WHEN is_resolved THEN incident_id ELSE NULL END) AS successful_resolutions,
        s.frequency
    FROM
        (SELECT DISTINCT incident_id, is_resolved FROM incidents WHERE is_resolved = true) as resolved_incidents
    JOIN
        signature_resolutions s ON resolved_incidents.incident_id = s.incident_id AND resolved_incidents.resolution = s.resolution
    GROUP BY s.signature, r.resolution
),
ranked_resolutions AS (
    SELECT
        signature,
        resolution,
        successful_resolutions / frequency AS success_rate,
        RANK() OVER (PARTITION BY signature ORDER BY successful_resolutions DESC) as rank
    FROM
        resolution_successes
)
SELECT * FROM ranked_resolutions WHERE rank = 1;

This query works as follows:

The signature_resolutions CTE groups incidents by their signatures and resolutions, counting the frequency of each.
The resolution_successes CTE joins resolved incidents with our previous result to count how many times a resolution was successful for each signature.
Finally, we rank these resolutions within each signature based on success rate.

**Comparison:**

| signature | resolution   | frequency | successful_resolutions | success_rate | rank |
|-----------|--------------|----------|------------------------|--------------|------|
| SIG-01    | Restart      | 100      | 85                     | 0.85         | 1    |
|          | Reboot       | 90       | 72                     | 0.80         | 2    |
|          | Update       | 60       | 48                     | 0.80         | 3    |

**Code:**

```sql
WITH signature_resolutions AS (
    SELECT
        signature,
        resolution,
        COUNT(*) AS frequency
    FROM incidents
    GROUP BY signature, resolution
),
resolution_successes AS (
    SELECT
        s.signature,
        r.resolution,
        COUNT(DISTINCT CASE WHEN is_resolved THEN incident_id ELSE NULL END) AS successful_resolutions,
        s.frequency
    FROM
        (SELECT DISTINCT incident_id, is_resolved FROM incidents WHERE is_resolved = true) as resolved_incidents
    JOIN
        signature_resolutions s ON resolved_incidents.incident_id = s.incident_id AND resolved_incidents.resolution = s.resolution
    GROUP BY s.signature, r.resolution
),
ranked_resolutions AS (
    SELECT
        signature,
        resolution,
        successful_resolutions / frequency AS success_rate,
        RANK() OVER (PARTITION BY signature ORDER BY successful_resolutions DESC) as rank
    FROM
        resolution_successes
)
SELECT * FROM ranked_resolutions WHERE rank = 1;
```

