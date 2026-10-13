---
type: "practice"
difficulty: "Expert"
---

# How do you decide the org-wide policy on whether AIOps-driven auto-rightsizing and auto-scaling actions may consider your commitment coverage as an input — i.e., the platform preferring to scale onto already-committed capacity? Walk me through the risk of the automation stranding or over-buying commitments as it optimizes.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Consolidating Cost Tooling](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Consolidating_Cost_Tooling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When deciding whether to incorporate commitment coverage into AIOps-driven auto-rightsizing and auto-scaling policies, we need to balance risk against benefits. The key is understanding the trade-offs between stranded commitments, over-buying, and the overall cost optimization goals of our platform.

#### Triaging the Question

The core question here involves balancing resource utilization with financial commitment management in an AIOps environment. The policy decision needs to ensure that automated scaling actions are aligned with long-term strategic objectives related to cloud spend governance. Specifically, we need to avoid over-buying commitments while also preventing underutilization and stranded resources.

#### Assumptions
Commitment Model: There is a commitment-based pricing model in place where some resources are reserved for specific periods.
AIOps Capabilities: The platform has sophisticated AIOps capabilities, including automated rightsizing and scaling.
Financial Metrics: Key financial metrics such as cost per unit of resource, committed vs. uncommitted spend, and overall budget constraints are available.
Step-by-Step Solution:
1. Define the Problem

The primary goal is to ensure that AIOps-driven actions do not lead to stranded commitments or over-buying in scenarios where scaling decisions rely on commitment coverage. Stranded commitments occur when resources reserved for future periods remain underutilized, while over-buying happens when more resources are committed than needed.

2. Evaluate Risk Factors
Stranding Risks: Overly aggressive rightsizing could lead to unused commitments.
Over-Buying Risks: Scaling too conservatively might result in missed cost-saving opportunities or over-purchasing of commitments.
3. Define Metrics and Thresholds

To balance these risks, we need to define:

Utilization Targets: Set target utilization levels for each resource type (e.g., CPU, memory) that the platform aims to achieve.
Commitment Utilization Targets: Establish thresholds for how much of a commitment should be utilized before scaling actions kick in.
Spend Metrics: Monitor cost per unit and total committed vs. uncommitted spend.
4. Implement Decision Logic

The decision logic should incorporate these metrics:

PYTHON
def decide_scaling_action(current_utilization, target_utilization, current_spend, max_spend):
    if current_utilization < target_utilization and current_spend + new_cost <= max_spend:
        return "scale in"
    elif current_utilization > target_utilization * 1.5: # Over-buying threshold
        return "scale out"
    else:
        return "no action"

# Example usage:
current_utilization = 0.7
target_utilization = 0.8
current_spend = 200
max_spend = 300

print(decide_scaling_action(current_utilization, target_utilization, current_spend, max_spend))
5. Monitor and Adapt

Continuous monitoring is crucial to adapt the policy based on changing conditions:

Regular Audits: Conduct regular reviews of commitment usage versus actual spend.
Feedback Loops: Use telemetry data from AIOps actions to refine thresholds over time.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Incorporating commitment coverage into auto-rightsizing and auto-scaling policies requires a nuanced approach. We must ensure that the platform scales intelligently while respecting long-term financial commitments. Key mistakes include overly simplistic threshold setting, ignoring real-time cost trends, and not adapting to changing workloads.

#### Follow-up
questions might explore specific scenarios or edge cases, such as:

How would you handle sudden spikes in demand?
What measures can be taken to prevent over-buying during periods of low usage?

