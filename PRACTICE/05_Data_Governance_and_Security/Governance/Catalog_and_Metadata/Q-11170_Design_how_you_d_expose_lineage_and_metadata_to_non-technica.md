---
type: "practice"
difficulty: "Deep"
---

# Design how you'd expose lineage and metadata to non-technical stakeholders (auditors, compliance, execs) versus engineers. What's the right abstraction and UI so each audience gets a trustworthy, non-overwhelming view?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Cataloging External Data](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Cataloging_External_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an effective way of exposing lineage and metadata to both technical and non-technical stakeholders, we need to consider their varying levels of expertise and the type of information they require. The key is to provide a tailored view that balances depth with usability. For non-technical audiences like auditors, compliance teams, and executives, the focus should be on high-level insights and clear visualizations. Engineers, on the other hand, need more detailed data lineage and metadata.

#### Triaging the Question

The core of this question is about creating an abstraction layer that can adapt to different levels of technical proficiency while ensuring trustworthiness. We will assume we have a robust data catalog system in place with comprehensive lineage and metadata information. The challenge is designing user interfaces (UIs) that serve both audiences effectively.

#### First-Principles Solution
1. High-Level View for Non-Technical Stakeholders

For non-technical stakeholders, the primary goal is to provide a clear and concise view of how data flows through the system without overwhelming them with technical details. This can be achieved by creating a high-level summary dashboard.

Dashboard Layout: The dashboard should include key metrics such as:

Data sources
Transformation steps
Sinks (data destinations)
Time taken for transformations
Compliance status

Visualizations: Use intuitive visualizations like flow diagrams, pie charts, and bar graphs to represent data lineage. For example, a flow diagram can show the journey of data from source to destination with key transformation steps highlighted.

2. Detailed View for Engineers

Engineers need more granular information to understand the complexities of the system. A detailed view should include:

Lineage Traces: Provide traceability through lineage graphs that show the complete history of transformations.
Transformation Details: Include details like schema changes, data manipulations, and business rules applied during processing.
Error Logs & Metrics: Display logs for failed or slow processes, along with metrics on system performance.
3. Abstraction Layers

To ensure trustworthiness and usability:

Audit Trails: Implement immutable audit trails that record all changes made to metadata and lineage information.
Role-Based Access Control (RBAC): Ensure access controls are in place so only relevant users can view specific levels of detail.
4. User Interface Design

Design the UI with user personas in mind:

Auditors & Compliance Teams: A simpler, more visual dashboard that highlights key compliance metrics.
Executives: Dashboards that provide a high-level overview and strategic insights without technical jargon.
Engineers: More detailed views with tools for deep dive analysis.

#### Closing Summary

By designing tailored UIs based on user personas, we can ensure that both non-technical stakeholders and engineers receive the information they need. Non-technical users get a clear, concise view, while engineers have access to detailed metadata and lineage information. The key is to balance depth with usability through appropriate abstractions.

#### Common Mistakes
& Follow-Ups

#### Common mistakes
include designing interfaces that are too complex for non-technical stakeholders or too simplistic for engineers, leading to miscommunication and reduced trust in the system. It’s important to conduct user testing and gather feedback to ensure the design meets the needs of all users effectively.

**Comparison:**

```
Dashboard Type	Features	Target Audience
Non-Technical	- High-level lineage summary<br>- Compliance metrics<br>- Visualizations	Auditors, Compliance Teams, Executives
Engineers	- Detailed lineage traces	Data Scientists, ETL Developers
```

**Example input / output:**

```

Example Input Output for Auditors Dashboard

Input: Query to fetch data lineage for a specific dataset.
Output: Visualized flow diagram showing data sources, transformations, and destinations.
JSON
Copy
{
  "source": "Sales Data",
  "transformations": [
    {
      "step": "Filtering by Date Range",
      "timeTaken": "30 seconds"
    },
    {
      "step": "Aggregating Sales Totals",
      "timeTaken": "1 minute"
    }
  ],
  "destination": "Sales Aggregated Data"
}

Example Input Output for Engineers Dashboard

Input: Query to trace data lineage through multiple transformations.
Output: Detailed step-by-step lineage graph with schema changes and business rules.
JSON
Copy
{
  "source": "Raw Sales Data",
  "transformations": [
    {
      "step": "Filtering by Date Range (2023-01-01 to 2023-12-31)",
      "timeTaken": "30 seconds"
    },
    {
      "step": "Aggregating Sales Totals",
      "timeTaken": "1 minute"
    }
  ],
  "destination": "Sales Aggregated Data",
  "schemaChanges": [
    "Changed column type from string to integer for quantity field."
  ]
}
```

**Flow:**

1. User Selects Dashboard
2. Auditors & Compliance
3. Fetched Lineage Summary
4. User Selects Dashboard
5. Engineers
6. Detailed Lineage Traces

