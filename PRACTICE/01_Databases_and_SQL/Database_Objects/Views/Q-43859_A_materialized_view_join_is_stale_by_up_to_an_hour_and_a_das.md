---
type: "practice"
difficulty: "Deep"
---

# A materialized view join is stale by up to an hour and a dashboard shows wrong totals during that window; how do you communicate and design around this staleness contract?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a materialized view (MV) that joins tables and remains stale by up to an hour, leading to inaccurate dashboard totals, we need to communicate effectively with stakeholders and design a solution that respects this staleness contract. This involves understanding the underlying issues, communicating clearly, and designing a resilient system.

Firstly, let’s clarify what is happening: materialized views are precomputed queries that store results in a database table for faster access. However, if the MV joins multiple tables and these tables change frequently, there can be a delay before the MV updates. In this case, the dashboard might show incorrect data during the hour-long staleness window.

#### Triaging the Question

The key issues here are:

Understanding the Stakeholders' Needs: The dashboard is likely critical for business operations or decision-making processes.
Communication: We need to communicate the staleness contract clearly and proactively inform stakeholders about potential inaccuracies.
Designing Around Staleness: Develop strategies to either minimize the impact of staleness or provide clearer feedback to users.

#### Assumptions
The MV is designed for performance optimization, not real-time updates.
The dashboard depends on data freshness but cannot handle frequent queries due to performance constraints.
There are no other ways to access up-to-date data in real time within the application.

#### First Principles Solution

To manage this situation, we need a multi-pronged approach:

Communicate Clearly: Inform stakeholders about the staleness contract and its implications.
Implement Clear Indicators: Provide visual cues on the dashboard indicating when data is stale.
Cache Management: Ensure that cache invalidation policies account for the staleness window.
Communicating with Stakeholders

Step 1: Identify Key Stakeholders Stakeholders include business analysts, developers, and end-users who interact with the dashboard.

Step 2: Prepare a Clear Message Create a message explaining:

What is happening: Materialized views can be stale for up to an hour.
Impact on Usage: Dashboards will show potentially inaccurate data during this window.
Resolution Strategy: We are working on strategies to mitigate the impact.

#### Example
Communication:

CODE
Subject: Materialized View Staleness Update

Dear Team,

We have identified that our materialized view for dashboard X is up to an hour stale, leading to potential inaccuracies. To manage expectations and ensure we address this effectively:

- The MV will be updated every hour.
- During the update window (last hour), dashboards may show inaccurate data.
- We are working on strategies to reduce this impact or provide clearer indicators.

Thank you for your understanding as we work through this.

Best,
[Your Name]
Implementing Clear Indicators

Step 1: Add Staleness Indicators Update the dashboard to include a visual indicator (e.g., a badge) showing when data is stale. This helps users avoid making critical decisions based on potentially inaccurate information.

HTML
<!-- Example HTML for Dashboard -->
<div class="dashboard">
    <h1>Dashboard Title</h1>
    <!-- Other Dashboard Content -->
    <div class="staleness-indicator">Data Stale (Last updated: [Timestamp])</div>
</div>

Step 2: Update Cache Management Ensure that any cache management systems are configured to invalidate the MV within an hour. This prevents users from accessing stale data via cached results.

SQL
-- Example SQL for Cache Management
CREATE OR REPLACE MATERIALIZED VIEW sales_summary AS
SELECT ... -- Materialized View Definition

-- Invalidate Cache After One Hour
ALTER TABLE sales_summary INVALIDATE LIFECYCLE 1;
Designing Around Staleness

Step 3: Prioritize Critical Queries Identify critical queries that should always use the most up-to-date data and ensure these are executed outside the MV staleness window.

SQL
-- Example Query for Up-To-Date Data
SELECT * FROM sales_summary WHERE timestamp > (CURRENT_TIMESTAMP - INTERVAL '1 hour');

Step 4: Use Real-Time Data Sources If feasible, provide a secondary real-time data source that users can fall back on during the MV staleness window. This could be a streaming service or a lightweight API.

JSON
{
  "service": "real_time_sales_api",
  "url": "http://api.example.com/sales"
}

#### Closing Summary
and Common Mistakes

#### Common Mistakes
Failing to communicate the issue properly, ignoring user experience during staleness windows, and not implementing robust cache management.

#### Follow-Ups
Discuss potential long-term solutions like real-time MVs or hybrid approaches combining real-time data with precomputed summaries.
TL;DR

Communicate effectively with stakeholders about the MV staleness window, provide clear visual indicators on dashboards, and implement robust caching strategies to minimize impact. This ensures users remain informed and can make decisions based on accurate data within their acceptable time frame.

**Comparison:**

```
Time	Action
08:00	MV updated with fresh data.
09:00	Dashboard shows potentially stale data.
10:00	User sees staleness indicator on dashboard.
```

**Code:**

```

CREATE OR REPLACE MATERIALIZED VIEW sales_summary AS SELECT * FROM orders JOIN customers ON ... JOIN products ON ... WHERE timestamp > (CURRENT_TIMESTAMP - INTERVAL '1 hour');

ALTER TABLE sales_summary INVALIDATE LIFECYCLE 1;

CODE
Copy

```html
<div class="dashboard">
    <h1>Dashboard Title</h1>
    <!-- Other Dashboard Content -->
    <div class="staleness-indicator">Data Stale (Last updated: [Timestamp])</div>
</div>
```

**Example input / output:**

```text
Input:
- User accesses dashboard during the MV staleness window.

Output:
- Dashboard shows potentially inaccurate data.
- Visual indicator (staleness badge) appears on the dashboard.
- Users are informed about potential inaccuracies via a notification or help text.
```

**Data:**

```json
{
  "service": "real_time_sales_api",
  "url": "http://api.example.com/sales"
}
```

**Flow:**

1. Yes
2. No
3. User Accesses Dashboard
4. Is Data Stale?
5. Show Stale Data with Indicator
6. Notify User of Inaccuracies
7. Display Fresh Data

