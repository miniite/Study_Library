---
type: "practice"
difficulty: "Deep"
---

# How do you decide when a workload should leave AKS for Azure Container Apps or Functions to reduce operational burden, and how do you communicate that platform-tier decision to 50 teams?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ AKS and Container Platform Standards](<../../../../TOOLS/Azure/Azure_Platform/Azure_Functions_and_Cosmos_DB_Tuning/AKS_and_Container_Platform_Standards.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding whether to move a workload from Azure Kubernetes Service (AKS) to Azure Container Apps or Azure Functions, we aim to reduce operational burden by leveraging managed services with lower overhead and easier management. The key decision points involve evaluating the application's requirements, current infrastructure setup, cost-benefit analysis, and long-term strategic considerations. This process involves assessing factors such as scalability needs, deployment complexity, and team expertise.

To communicate this platform-tier decision effectively to 50 teams, we need a clear, structured approach that ensures understanding, buy-in, and support across the organization. Here’s how you can systematically evaluate and implement this change:

Evaluating Application Requirements:

Scalability: Assess if the application needs horizontal or vertical scaling capabilities provided by Azure Container Apps.
Complexity: Determine whether the workload's complexity justifies using a more flexible platform like AKS, or if simpler managed services can handle it.
Security and Compliance: Check if there are specific security or compliance requirements that might necessitate an on-premises Kubernetes deployment.

Cost-Benefit Analysis:

Calculate the ongoing costs of running the application in AKS versus Azure Container Apps or Functions, including maintenance, operational overhead, and potential savings.

#### Cons
ider the total cost of ownership (TCO) over time for each platform.

Operational Burden Reduction:

Evaluate how much operational effort can be shifted to managed services without compromising performance or availability.
Measure the reduction in manual intervention required by moving to a more automated service.

Long-Term Strategic Vision:

Align the decision with overall cloud strategy and technology roadmap.
Ensure that the chosen platform supports future growth and innovation needs of the application.

Communication Strategy:

Develop a comprehensive communication plan, involving key stakeholders from each team.
Use clear, concise language to explain why this change is beneficial and how it will impact different teams.
Provide training sessions or documentation to help teams understand and adopt new services effectively.
Set expectations regarding timelines, milestones, and support resources.

#### Implementation
Plan:

Create a phased implementation plan that minimizes disruption while maximizing benefits.
Prioritize workloads based on their criticality and ease of migration.
Leverage automation tools and pipelines to streamline the transition process.

Monitoring and Feedback Loops:

Set up monitoring for performance, cost, and operational health post-migration.
Establish feedback channels where teams can report issues or suggest improvements.
Continuously refine the approach based on real-world data and team input.

**Comparison:**

```
Platform	Monthly Cost	Annual Ops Effort (Hrs)
AKS	$5000	240
Container Apps	$1000	60
Team	Priority Level	Reason for Migration
DevOps	High	Reduced ops burden, easier management
App Development	Medium	Simplified deployment, reduced complexity
Analytics	Low	No significant impact on current operations
```

**Code:**

```
Example of a simple cost-benefit analysis script
PYTHON
Copy
def calculate_cost_benefit(platform, workload_details):
    if platform == "AKS":
        # Placeholder for AKS costs
        monthly_cost = 5000
        ops_effort = 20
    elif platform == "Container Apps":
        monthly_cost = 1000
        ops_effort = 5

    current_ops_cost = workload_details["current_ops"]
    future_ops_savings = ops_effort * 12

    net_savings = (future_ops_savings - monthly_cost) / 12

    return {
        "monthly_cost": monthly_cost,
        "net_savings": net_savings
    }
```

**Example input / output:**

```text
# Example input and output for the cost-benefit analysis

workload = {
    "current_ops": 300,
    "name": "Sample Workload",
    "required_scalability": "horizontal",
    "complexity_level": "low"
}

result = calculate_cost_benefit("Container Apps", workload)
print(result) # Output: {'monthly_cost': 1000, 'net_savings': -25.0}
```

**Data:**

```json
{
  "workload_details": {
    "current_ops": 300, // Annual operations cost in dollars
    "name": "Sample Workload",
    "required_scalability": "horizontal",
    "complexity_level": "low"
  }
}
```

**Flow:**

1. Yes
2. No
3. Decision Point 1: EvaluateApplication Requirements
4. Is the workload suitable for amanaged service?
5. Decision Point 2: Cost-BenefitAnalysis
6. Migrate to ContainerApps/Functions
7. Decision Point 3: Long-TermStrategic Vision
8. Consider AKS for complex,scalable workloads

