---
type: "practice"
difficulty: "Expert"
---

# Present a build-vs-buy analysis for self-hosted Vault OSS versus HCP Vault Dedicated versus AWS Secrets Manager as your org-wide platform. What TCO and lock-in factors drive the recommendation?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Keeping Secrets out of Env and Logs](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Keeping_Secrets_out_of_Env_and_Logs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this build-vs-buy analysis, we will explore the self-hosted HashiCorp Vault Open Source Software (OSS) versus HashiCorp Enterprise Cloud Platform (HCP) Vault Dedicated and AWS Secrets Manager as a platform for secrets management across an organization. The key factors to consider are total cost of ownership (TCO), lock-in, security, ease of deployment, and ongoing maintenance. We will evaluate each option based on these criteria.

**Comparison:**

| Option            | TCO                  | Lock-In        | Security                | Ease of Deployment | Maintenance |
|-------------------|----------------------|----------------|-------------------------|--------------------|-------------|
| Self-Hosted Vault OSS | Low (internal resources) | None         | High with proper config | Moderate          | Moderate    |
| HCP Vault Dedicated | Moderate (initial + ongoing support) | High       | High with built-in features | Low               | Low         |
| AWS Secrets Manager | Moderate (AWS costs + support) | Medium to high | High (compliance, security) | Low                | Low         |

**Code:**

```
MERMAID
Copy
graph TB
    A[Self-Hosted Vault OSS] --> B1(TCO)
    A --> C(lock-in)
    A --> D(security)
    A --> E[ease of deployment]
    A --> F[maintenance]

    HCP Vault Dedicated --> G1(TCO)
    HCP Vault Dedicated --> H(lock-in)
    HCP Vault Dedicated --> I(security)
    HCP Vault Dedicated --> J[ease of deployment]
    HCP Vault Dedicated --> K[maintenance]

    AWS Secrets Manager --> L1(TCO)
    AWS Secrets Manager --> M(lock-in)
    AWS Secrets Manager --> N(security)
    AWS Secrets Manager --> O[ease of deployment]
    AWS Secrets Manager --> P[maintenance]
```

**Example input / output:**

| Option            | TCO                  | Lock-In        | Security                | Ease of Deployment | Maintenance |
|-------------------|----------------------|----------------|-------------------------|--------------------|-------------|
| Self-Hosted Vault OSS | Low (internal resources) | None         | High with proper config | Moderate          | Moderate    |
| HCP Vault Dedicated | Moderate (initial + ongoing support) | High       | High with built-in features | Low               | Low         |
| AWS Secrets Manager | Moderate (AWS costs + support) | Medium to high | High (compliance, security) | Low                | Low         |

**Data:**

```json
{
  "Self-Hosted Vault OSS": {
    "TCO": "Low, but requires significant internal resources for setup and maintenance",
    "lock-in": "None, as it's open source and can be migrated easily to other platforms if needed",
    "security": "High with proper configuration, but may require custom integrations and monitoring",
    "ease_of_deployment": "Moderate, needs DevOps expertise"
  },
  "HCP Vault Dedicated": {
    "TCO": "Moderate, initial setup cost + ongoing maintenance by HashiCorp support",
    "lock-in": "High, due to specialized integration and potentially unique features only available in HCP",
    "security": "High with built-in security features, but may require additional configuration",
    "ease_of_deployment": "Low, as it's fully managed"
  },
  "AWS Secrets Manager": {
    "TCO": "Moderate, low initial cost + ongoing AWS usage costs and support fees",
    "lock-in": "Medium to high, due to AWS ecosystem integration and potential for increased dependencies",
    "security": "High with robust built-in security features and compliance certifications",
    "ease_of_deployment": "Low, needs DevOps expertise"
  }
}
```

**Flow:**

1. AWS Secrets Manager
2. AWS Costs
3. AWS Management Console
4. AWS Integration
5. Medium to High Lock-In
6. Security Features
7. TCO
8. HCP Vault Dedicated
9. Initial Setup Cost
10. HashiCorp Support
11. Managed Environment
12. Low Ongoing Effort
13. High Lock-In
14. TCO
15. Self-Hosted Vault OSS
16. Setup Cost
17. Initial DevOps Effort
18. Ongoing Maintenance
19. TCO

