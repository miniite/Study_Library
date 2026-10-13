---
type: "practice"
difficulty: "Expert"
---

# A bar-raiser asks you to reconcile FinOps with platform-engineering's 'golden path' philosophy: if the paved road is cheapest by construction, is a separate FinOps optimization function even necessary? Argue where the two overlap and where they must stay distinct.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Scaling Cost Governance](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Scaling_Cost_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To reconcile FinOps with platform-engineering's 'golden path' philosophy, we need to understand that both serve different yet interconnected purposes. The golden path focuses on standardizing and optimizing the infrastructure deployment process to ensure consistency, security, and cost efficiency. On the other hand, FinOps emphasizes the alignment of financial practices with operational decisions to maximize cost savings and resource utilization.

The overlap between these two concepts lies in their shared goal of reducing costs while maintaining service quality. However, they must remain distinct as they operate at different levels: infrastructure standardization vs. financial optimization.

Firstly, let's break down what 'golden path' means. It involves setting up standardized environments, using consistent tools and templates to deploy applications, and ensuring that all resources are deployed in a way that minimizes overhead costs. This ensures that the foundation is laid efficiently and cost-effectively for any development team to build upon.

Secondly, FinOps takes this a step further by integrating financial oversight into the entire lifecycle of cloud usage. It involves monitoring and analyzing cloud spend, identifying areas where costs can be optimized, and ensuring that spending aligns with business objectives. This approach requires insights into not just how resources are deployed but also why certain resources were chosen and at what cost.

Now, let's delve deeper into the overlap and distinct aspects of these two concepts:

Overlap:
Cost Optimization: Both aim to reduce costs by standardizing infrastructure deployment and monitoring resource usage.

#### Cons
istency and Compliance: Golden path ensures that all teams follow a consistent set of best practices, which can be enforced through policies in FinOps as well.
Distinct Aspects:

Scope:

Golden Path: Focuses on the initial setup and deployment process of applications. It aims to create an efficient foundation for development.
FinOps: Operates throughout the lifecycle of cloud resources, from procurement to decommissioning. It involves continuous monitoring and optimization.

Tools and Practices:

Golden Path: Utilizes tools like Terraform or CloudFormation templates for consistent deployments. This is more about infrastructure as code.
FinOps: Leverages cost management consoles, budget alerts, and detailed reports to track and optimize cloud spend. It involves data analytics and financial modeling.

Decision-Making:

Golden Path: Ensures that the deployment process is streamlined and secure. Decisions here are often technical in nature.
FinOps: Involves strategic decisions about which resources to use, how much to budget, and whether to scale up or down based on business needs.

#### Example Scenario

#### Cons
ider a scenario where a development team is building a new application. The golden path would focus on setting up the infrastructure using standardized templates, ensuring that all instances are deployed securely and efficiently. However, FinOps would come into play when monitoring this deployment over time. If it detects that certain resources are underutilized or if there’s an unexpected spike in costs, it could recommend reallocation of those resources to more efficient ones.

#### Common Mistakes
Overlapping Tools: Combining tools meant for infrastructure setup (like Terraform) with cost optimization (like CloudWatch) can lead to confusion and inefficiency.
Ignoring Long-Term Costs: The golden path might focus on initial deployment costs but not long-term operational expenses, which FinOps would address.

#### Conclusion

While the golden path provides a solid foundation for efficient infrastructure deployment, FinOps ensures that this setup is continuously optimized for financial efficiency. Both are crucial components in achieving overall cost governance and should be integrated seamlessly to maximize benefits.

**Code:**

```
MERMAID
Copy
graph LR;
    A[Golden Path] -->|Initial Setup| B[Standardized Templates];
    B --> C[Consistent Deployments];
    A --> D[Financial Efficiency];
    D --> E[Cost Monitoring];
```

**Flow:**

1. Infrastructure Setup
2. Consistency & Security
3. Standardized Templates
4. Cost Optimization
5. Financial Oversight

