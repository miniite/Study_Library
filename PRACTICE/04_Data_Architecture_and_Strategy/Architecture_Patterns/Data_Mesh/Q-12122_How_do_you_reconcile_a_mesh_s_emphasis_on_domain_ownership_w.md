---
type: "practice"
difficulty: "Deep"
---

# How do you reconcile a mesh's emphasis on domain ownership with the reality that some critical data crosses every domain and needs enterprise stewardship?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Domain Ownership vs Cross-cutting Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Mesh/Domain_Ownership_vs_Cross-cutting_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconcile a Data Mesh’s emphasis on domain ownership with the reality that some critical data crosses multiple domains, we need to establish a balance between localized data governance and centralized stewardship. The core idea is to leverage both local control and enterprise-wide coordination to ensure consistent quality, compliance, and usability of shared data.

#### Triage

The question delves into the tension between decentralization (domain ownership) and centralization (enterprise stewardship). The goal is to design a framework that allows for both autonomy in managing domain-specific data while ensuring that cross-domain critical data remains governed effectively. Key assumptions include:

Each domain has unique data needs but may also share essential datasets.
Centralized governance can provide standards, security policies, and compliance mechanisms.
Local teams are best positioned to understand and manage their specific data.

#### First Principles Solution
Domain Data Owners (DDOs): Each domain should have a designated owner who is responsible for the data within that domain.
Cross-Domain Data Stewards: Appoint enterprise-level stewards or cross-functional teams to oversee critical, shared datasets.
Data Platform Architecture: Design a hybrid platform architecture that supports both localized and centralized aspects.
Governance Frameworks: Implement governance frameworks that combine local policies with overarching rules for cross-domain data.
Step-by-Step Solution

Identify Cross-Domain Data:

Review the critical datasets that are used across multiple domains to determine which ones require special stewardship.

Establish DDO Roles and Responsibilities:

Each domain should have a clear owner responsible for maintaining data quality, security, and compliance within their domain.

Form Cross-Domain Steward Teams:

Assemble teams from relevant domains that are responsible for overseeing the critical shared datasets.
These teams can help enforce data standards, ensure consistency, and provide guidance on cross-domain use cases.

Develop Centralized Governance Policies:

Create policies and procedures that outline how cross-domain data should be managed, accessed, and updated.
Ensure these policies are aligned with broader organizational goals related to compliance and security.

Implement Technical Infrastructure:

Design a platform architecture that supports both local and shared datasets.
Use tools like metadata repositories, data cataloging systems, and API gateways to facilitate cross-domain access while maintaining governance controls.

Regular Audits and Reviews:

Conduct periodic audits to ensure compliance with both domain-specific and enterprise-wide policies.
Foster a culture of transparency and collaboration among all stakeholders involved in shared datasets.

#### Closing Summary

In summary, reconciling the emphasis on domain ownership with the need for enterprise stewardship involves creating a balanced framework that leverages local expertise while ensuring cross-domain data is managed effectively. This approach requires clear roles, robust governance policies, and appropriate technical infrastructure to support both localized and centralized needs.

#### Common Mistakes/Follow-Ups
Ignoring Local Expertise: Over-centralizing control can undermine the value of domain-specific knowledge.
Lack of Clarity in Roles: Ambiguity about who owns or governs shared data can lead to inconsistencies.
Inadequate Technical Support: Without proper tools and infrastructure, maintaining governance across domains becomes challenging.

Would you like a deeper dive into any specific aspect of this solution? For example, how we could implement metadata management or API governance?

**Comparison:**

```
Domain	Owner	Shared DataSets
Sales	Alice
Finance	Bob	Customer Transactions
Cross-Domain Steward Team Responsibilities
Ensure consistency and compliance of shared data.
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Domain Data Owner (DDO)] --> B{Is the data shared with other domains?}
    B -- Yes --> C(Cross-Domain Steward Team)
    B -- No --> D[Local Data Management]
    C --> E[Implement Cross-Functional Policies and Standards]
    E --> F[Regular Audits and Reviews]
```

**Example input / output:**

```

Input:

Domain A shares critical dataset X with Domain B
Domain B has a domain owner (Bob) but no dedicated cross-domain steward

Output:

Bob is appointed as the initial cross-domain steward for dataset X.
The Data Governance Team reviews and finalizes policies for dataset X to ensure both domains are aligned.
Regular audits are scheduled every quarter.

Request: Implementing a metadata repository for managing shared datasets.

Response:

JSON
Copy
{
  "metadataRepository": {
    "datasetX": {
      "owners": ["Bob"],
      "stewards": ["Data Governance Team"],
      "policies": [
        {"name": "Compliance Policy", "version": 1.0},
        {"name": "Access Control Policy", "version": 2.0}
      ]
    }
  }
}
```

**Data:**

```json
{
  "domains": [
    {
      "name": "Sales",
      "owner": "Alice",
      "sharedDataSets": []
    },
    {
      "name": "Finance",
      "owner": "Bob",
      "sharedDataSets": ["Customer Transactions"]
    }
  ],
  "crossDomainStewards": [
    {
      "name": "Data Governance Team",
      "domainsInvolved": ["Sales", "Finance"],
      "responsibilities": ["Ensure consistency and compliance of shared data"]
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Domain A
4. Identify Shared Dataset X
5. Audit Data Ownership andGovernance Needs
6. Assign Domain Owner (Bob)
7. No Cross-Domain StewardNeeded
8. Appoint Cross-DomainSteward Team
9. Implement Policies andStandards
10. Conduct Regular Audits

