---
type: "practice"
difficulty: "Expert"
---

# How do you decide between a hub-and-spoke landing zone and a decentralized account-per-team model when onboarding 200 engineering teams onto AWS Organizations?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Platform vs Product Ownership](<../../../../06_Cloud_Computing/Cloud_Architecture/Architecture_Strategy/Platform_vs_Product_Ownership.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between a hub-and-spoke landing zone and a decentralized account-per-team model when onboarding 200 engineering teams onto AWS Organizations, you need to consider several key factors including governance, security, cost, complexity, and scalability. The goal is to strike the right balance that aligns with your organization's strategic objectives.

First, let’s triage the question:

What are we comparing? Hub-and-spoke landing zone vs. decentralized account-per-team.
What is being probed? Governance, security, cost, and overall manageability of cloud resources.

Now, step by step, from first principles:

1. Hub-and-Spoke Landing Zone Model

This model involves creating a centralized hub organization within AWS Organizations. Each engineering team has its own spoke account that connects to the central hub. The hub is responsible for central governance and control plane operations like billing, security, and compliance.

Key Considerations:
Governance: Centralized policies are easier to enforce.
Security: You can implement robust security measures at the hub level without having to configure them in every spoke account.
Cost: Can optimize costs by leveraging AWS services more efficiently through centralized management.
Complexity: Initial setup and ongoing maintenance can be complex, especially with 200 teams.

#### Example
Policies
Policies
Policies
Hub Organization
Spoke A
Spoke B
Spoke C
Hub
2. Decentralized Account-Per-Team Model

This model involves creating an individual AWS account for each engineering team, with minimal central governance.

Key Considerations:
Governance: Each team has full autonomy and can manage their own security and compliance.
Security: Security policies need to be duplicated in each account, which can lead to inconsistencies.
Cost: Potentially less cost-efficient as you might not leverage centralized billing or other organizational benefits.
Complexity: Simplifies the initial setup but increases complexity in terms of monitoring and managing a large number of accounts.

#### Example
FLOWCHART · MERMAID
graph TB
    TEAMA[Team A]
    TEAMB[Team B]
    TEAMC[Team C]

    TeamA -- Direct --> AWS Account A
    TeamB -- Direct --> AWS Account B
    TeamC -- Direct --> AWS Account C
3. Strategic Decision Criteria

Given the scale (200 teams), here are some strategic criteria to evaluate both models:

Governance: The need for centralized governance policies and control is critical.
Security: Ensuring consistent security measures across all accounts requires significant effort if using a decentralized model.
Cost Efficiency: Centralized management can lead to cost savings, especially with shared services and efficient resource utilization.
Team Autonomy vs. Control: Balancing the autonomy teams need for their projects against the centralized control required for compliance.
4. Recommendation

Given that you have 200 engineering teams, a hub-and-spoke model is more suitable:

Governance and Security: Centralizing governance and security policies significantly reduces complexity and risk.
Cost Efficiency: Leveraging AWS Organizations can optimize costs by sharing services and resources efficiently.
Manageability: Easier to manage and audit with a centralized hub.

However, consider implementing a hybrid approach for smaller teams or specific projects that require more autonomy:

Hybrid Model: Allow certain small teams to use the decentralized model if they have a high level of trust and robust internal controls.
5. Conclusion

For the primary onboarding strategy, hub-and-spoke is recommended due to its ability to provide centralized governance and security while maintaining cost efficiency. However, flexibility in managing smaller or more autonomous teams can be accommodated through a hybrid approach.

#### Common Mistakes/Follow-ups
Overlooking the importance of consistent security policies.
Underestimating the complexity and effort required for initial setup and ongoing maintenance.
Ignoring the potential for increased costs with a decentralized model without robust governance measures in place.

**Comparison:**

```
Hub-and-Spoke Model	Decentralized Model
Central governance	No central control
Easier to manage	More complex
Cost-efficient	Potentially higher costs
Consistent security	Inconsistent
```

**Code:**

```python
# Example Python code to simulate account creation (simplified)
def create_hub_and_spoke_accounts(num_teams):
    hub = "CentralHubOrg"
    spoke_accounts = [f"Spoke{i}" for i in range(1, num_teams+1)]
    return {"hub": hub, "spokes": spoke_accounts}
```

**Example input / output:**

```

Input:

Number of engineering teams = 200

Output:

Hub Organization: CentralHubOrg
Spoke Accounts: [Spoke1, Spoke2, ..., Spoke200]

Dry-run Example:

PYTHON
Copy
create_hub_and_spoke_accounts(200)

Result:

PYTHON
Copy
{'hub': 'CentralHubOrg', 'spokes': ['Spoke1', 'Spoke2', /*...*/ 'Spoke200']}
```

**Data:**

```json
{
  "hub_org": "CentralHubOrg",
  "teams": [
    "TeamA", "TeamB", "TeamC", /* ... up to 200 teams */
    "TeamZ"
  ]
}
```

**Flow:**

1. Hub Organization
2. (Create Hub)
3. (Spokes Connect)
4. (Account Policies)
5. (Centralized Policies)
6. (Monitor and Control)
7. Decentralized Model
8. (Direct Teams)
9. (No Central Control)

