---
type: "practice"
difficulty: "Expert"
---

# How do you sunset a legacy 'golden' dataset that 400 downstream consumers depend on but that violates every modern governance standard? Design the deprecation, communication, and migration plan without breaking the business.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Layered Pipelines Bronze Silver Gold](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To sunset a legacy "golden" dataset in a way that minimizes disruption to 400 downstream consumers while ensuring compliance with modern governance standards, we need to approach this problem methodically. The core steps involve identifying dependencies, planning the transition, communicating effectively, and finally decommissioning the old system. Let's break down each step.

Firstly, it’s crucial to understand that this is not just a technical challenge but also an organizational one. We need to ensure smooth communication with stakeholders, manage risks, and plan for potential issues. The goal is to avoid any business disruption while ensuring adherence to modern data governance practices.

**Comparison:**

| Consumer | Current Data Source | Proposed Data Source |
|----------|---------------------|----------------------|
| SalesDepartment | Legacy Golden Dataset | New Data Warehouse   |
| MarketingTeam   | Legacy Golden Dataset | API Gateway          |
| FinanceTeam     | Legacy Golden Dataset | BI Dashboard         |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Identify Dependencies] --> B[Map Data Flow];
    C[Plan Transition] --> D[Design New ETL Pipeline];
    E[Migrate Consumers] --> F[Communicate Plan];
    G[Sunset Legacy System] --> H[Monitor and Maintain];
    I[Stakeholders] -- "Feedback" --> F;
    B -- "Dependencies" --> A;
```

**Example input / output:**

| Consumer | Current Data Source | Proposed Data Source |
|----------|---------------------|----------------------|
| SalesDepartment | Legacy Golden Dataset | New Data Warehouse   |
| MarketingTeam   | Legacy Golden Dataset | API Gateway          |
| FinanceTeam     | Legacy Golden Dataset | BI Dashboard         |

**Data:**

```yaml
dependencies:
  - consumer: SalesDepartment
    system: CRM
  - consumer: MarketingTeam
    system: EmailCampaigns
  - consumer: FinanceTeam
    system: BudgetAllocation
  ...
```

**Flow:**

1. Feedback
2. Dependencies
3. Identify Dependencies
4. Map Data Flow
5. Plan Transition
6. Design New ETL Pipeline
7. Migrate Consumers
8. Communicate Plan
9. Sunset Legacy System
10. Monitor and Maintain
11. Stakeholders

