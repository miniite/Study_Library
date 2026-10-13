---
type: "practice"
difficulty: "Expert"
---

# What's your framework for handling a data-breach involving training data — what governance artifacts must exist beforehand to scope impact within hours?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Governing Incident Data Reuse](<../../../../05_Data_Governance_and_Security/Governance/Data_Strategy/Governing_Incident_Data_Reuse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a data breach involving training data, it's crucial to have a robust and well-defined framework in place that can quickly scope the impact of the breach. My approach involves several key governance artifacts and steps to ensure rapid response. Here’s how I would handle such a situation:

Firstly, having a comprehensive Data Governance Framework in place is essential. This includes policies, procedures, and technical controls that govern data use, storage, and access. Specifically, we need three critical components:

Data Inventory: A detailed catalog of all training datasets, including metadata like purpose, origin, sensitivity level, and current status.
Access Control Policies: Defined roles and permissions for accessing different types of data. This ensures that only authorized personnel can handle sensitive information.
Incident Response Plan (IRP): A pre-defined set of actions to take in the event of a breach.

#### Triage
involves understanding what specific governance artifacts must be present to effectively scope the impact within hours. Let’s break down each step:

Step 1: Rapid Identification

The Data Inventory artifact is crucial for quickly identifying which datasets were involved in the breach. This requires a real-time, searchable catalog that can provide details on the dataset's location, version, and any recent modifications.

Step 2: Risk Assessment

Using the Access Control Policies, we can determine if unauthorized access occurred. This step involves cross-referencing user logs against the defined roles to identify potential breaches.

Step 3: Impact Analysis

The Incident Response Plan provides a structured approach for assessing the impact of the breach. Key steps include:

Data Classification: Determine which datasets were impacted based on their sensitivity level.
Scope of Data Exposure: Identify how much data was exposed and its potential impact on business operations or customer trust.
Compliance Obligations: Check if the breach affects compliance with any regulatory requirements.
Step 4: Mitigation Actions

Based on the risk assessment, mitigation actions are prioritized. This might include:

Data Masking/Encryption: Protect remaining data from further exposure.
Communication Plan: Develop a plan for informing stakeholders and customers about the breach.
Regulatory Reporting: Prepare necessary reports to regulatory bodies as required.
Step 5: Post-Incident Review

Finally, conduct a thorough post-breach review to improve governance processes. This includes revisiting policies, updating access controls, and enhancing overall data security measures.

In summary, having these governance artifacts in place ensures that the impact of a training data breach can be scoped within hours, minimizing the damage and allowing for swift recovery. Common pitfalls include inadequate documentation, lack of real-time monitoring capabilities, and insufficient cross-functional collaboration during response efforts.

**Comparison:**

```
Artifact	Description
Data Inventory	Catalogs all training datasets with metadata like purpose, origin, sensitivity level, and status.
Access Control	Defines roles and permissions for accessing different types of data.
IRP	Pre-defined set of actions to take in the event of a breach.
```

**Code:**

```text
# Example: Data Inventory Schema
data_inventory = {
  "dataset1": {"type": "training", "sensitivity": "high", "location": "AWS S3"},
  "dataset2": {"type": "testing", "sensitivity": "low", "location": "on-premises"}
}
```

**Example input / output:**

```text
Input (Example): A user with the role 'data_admin' accessed a high-sensitivity training dataset outside of working hours.
Output (Expected Response): The IRP is triggered, and steps are taken to identify which datasets were accessed, assess the risk, determine the scope of exposure, and implement mitigation actions.
```

**Data:**

```json
{
  "data_inventory": {
    "dataset1": {
      "type": "training",
      "sensitivity": "high",
      "location": "AWS S3"
    },
    "dataset2": {
      "type": "testing",
      "sensitivity": "low",
      "location": "on-premises"
    }
  },
  "access_policies": [
    {"user": "admin", "role": "data_admin"},
    {"user": "analyst1", "role": "data_reader"}
  ],
  "irp_steps": ["identification", "risk_assessment", "impact_analysis", "mitigation"]
}
```

**Flow:**

1. Yes
2. Data Breach Occurs
3. Is Data Inventory Available?
4. Identify Affected Datasets
5. Risk Assessment -Cross-Reference Logs withPolicies
6. Impact Analysis - DetermineScope & ComplianceObligations
7. Mitigation Actions - Protect,Communicate, Report
8. Post-Incident Review -Improve Governance Processes

