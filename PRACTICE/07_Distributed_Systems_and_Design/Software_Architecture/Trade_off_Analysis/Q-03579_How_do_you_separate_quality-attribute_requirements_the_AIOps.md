---
type: "practice"
difficulty: "Deep"
---

# How do you separate quality-attribute requirements the AIOps platform must meet from 'nice-to-have' capabilities teams merely want during utility-tree construction? Why does ATAM force this separation before analysis, and how does it stop the observability roadmap from becoming a dashboard-wishlist?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Utility Trees](<../../../../07_Distributed_Systems_and_Design/Software_Architecture/Trade_off_Analysis/Utility_Trees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, quality-attribute requirements (QARs) define what an AIOps platform must achieve to meet business and operational needs, while 'nice-to-have' capabilities are desired but not strictly necessary. ATAM forces this separation by methodically identifying and categorizing these attributes before beginning the utility-tree construction process. This ensures that the observability roadmap is grounded in essential functionality rather than falling prey to common pitfalls like wishlist-driven prioritization.

Let's break down the process from first principles:

Identify Quality-Attribute Requirements (QARs):

Business Goals: Align with business KPIs and service level objectives.
Operational Needs: Ensure the platform can detect, diagnose, and resolve issues efficiently.
User Expectations: Consider SLAs, MTTR, and other performance metrics.

Categorize 'Nice-to-Have' Capabilities:

These are often requested by teams who have specific use cases or preferences but do not directly impact the platform’s core functionality.

#### Example
s include custom dashboards, advanced alerting rules, or integration with niche tools.

Utility-Tree Construction:

The utility tree represents a hierarchical breakdown of the AIOps platform's components and their interdependencies.
Each node in the tree corresponds to an essential feature that contributes to meeting QARs.

ATAM Process:

Separation Stage: First, conduct workshops with stakeholders to clearly define QARs based on business priorities.
Categorization Stage: Categorize 'nice-to-have' capabilities separately.
Trade-Off Analysis: Analyze the costs and benefits of each feature, balancing them against QARs.

Roadmap Development:

Prioritize features based on their alignment with QARs and business value.
Ensure the roadmap is robust and not overly influenced by wishlist-driven requests that might divert focus from critical functionalities.

By following this approach, ATAM ensures that the observability roadmap focuses on essential capabilities that drive operational efficiency and reliability. This prevents the platform from becoming a collection of features requested by various teams without strategic alignment or prioritization.

**Comparison:**

| Requirement ID | Name                 | Category       |
|----------------|----------------------|----------------|
| 1              | Real-time alerting   | QAR            |
| 2              | Customizable dashboards | nice_to_have  |
| 3              | Automated root cause analysis | QAR        |

**Code:**

```text
# Example pseudo-code for QAR categorization
def categorize_requirements(requirements):
    qars = []
    nice_to_haves = []
    for req in requirements:
        if is_quality_attribute(req):
            qars.append(req)
        else:
            nice_to_haves.append(req)
    return qars, nice_to_haves

# Example function to check QAR
def is_quality_attribute(requirement):
    # Check business goals, operational needs, user expectations
    pass
```

**Example input / output:**

```

Input:

JSON
Copy
{
  "requirements": [
    {"id": 1, "name": "Real-time alerting", "category": "QAR"},
    {"id": 2, "name": "Customizable dashboards", "category": "nice_to_have"},
    {"id": 3, "name": "Automated root cause analysis", "category": "QAR"}
  ]
}

Output:

JSON
Copy
{
  "qars": [
    {"id": 1, "name": "Real-time alerting"},
    {"id": 3, "name": "Automated root cause analysis"}
  ],
  "nice_to_haves": [
    {"id": 2, "name": "Customizable dashboards"}
  ]
}
```

**Data:**

```json
{
  "requirements": [
    {"id": 1, "name": "Real-time alerting", "category": "QAR"},
    {"id": 2, "name": "Customizable dashboards", "category": "nice_to_have"},
    {"id": 3, "name": "Automated root cause analysis", "category": "QAR"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Are the requirements clear?
5. Identify and categorize QARs
6. Clarify requirements withstakeholders
7. Conduct ATAM to prioritizefeatures based on QARs
8. Categorize 'nice-to-have'capabilities
9. Develop utility-tree foressential components
10. Create observability roadmap

