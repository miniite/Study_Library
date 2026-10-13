---
type: "practice"
difficulty: "Expert"
---

# You want to centralize commitment purchasing so no single team over-commits, but teams complain the central pool 'steals' their savings. Design the internal RI/Savings-Plan sharing and rate-card model that keeps incentives aligned.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Spend Freezes and Prioritization](<../../../../06_Cloud_Computing/Cloud_Cost/FinOps/Serverless_Cost_Visibility/Spend_Freezes_and_Prioritization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of centralizing commitment purchasing while ensuring alignment with team incentives, we need to design a flexible, fair, and transparent Internal Rate-Card (IR) and Savings-Plan sharing model that balances overall cost efficiency with individual team savings potential.

#### Triage

The core issue is balancing financial optimization for the organization as a whole with the desire of teams to maximize their own savings. The key is creating an incentive structure where both central governance and team autonomy are respected. We need to ensure that savings plans can be shared fairly, and that teams have clear visibility into how costs are allocated.

#### First-Principles Solution
1. Define Clear Objectives
Centralize purchasing to achieve better negotiation leverage.
Ensure fairness in cost sharing among teams.
Align incentives by rewarding both overall savings and individual team performance.
2. Internal Rate-Card (IR) Model
Develop a dynamic IR that reflects current market rates, but allows for adjustments based on collective savings goals.
Implement a tiered pricing model where significant savings lead to lower rates across the board.
3. Savings Plan Sharing Mechanism
Create an allocation system where teams can submit proposals for additional commitments and share potential savings back into a common pool.
Establish a mechanism for transparently tracking and validating these savings.
4. Rate-Card Adjustment Logic
Regularly review the overall cost vs. savings to recalibrate rates.
Use a weighted average approach where central governance can override individual requests if they conflict with overall goals.

#### Implementation
Steps

Initial Setup:

Define initial IR based on current market rates and organizational spending patterns.
Create a shared dashboard for teams to track their usage, projected costs, and potential savings.

Savings Proposals:

Allow teams to propose additional commitments with expected savings.
Evaluate proposals using criteria such as cost impact, environmental benefits (if applicable), and alignment with overall goals.

Allocation and Validation:

Allocate approved savings back into the central pool for re-allocation or distribution based on performance metrics.
Use automated tools to validate and confirm cost reductions before adjustments are applied.

Review and Adjust Rates:

Conduct periodic reviews of the IR and Savings Plans, adjusting rates based on collective performance.
Ensure that any changes are communicated clearly to all teams to maintain transparency.

#### Closing Summary

The proposed model balances centralized governance with team incentives by dynamically adjusting rate cards and sharing savings through a transparent allocation process. This approach ensures that both the organization’s cost optimization goals and individual team savings are aligned, fostering collaboration rather than competition.

#### Common Mistakes/Follow-Ups
Over-Centralization: Teams may feel their autonomy is being restricted if the central pool is too rigid.
Lack of Transparency: Without clear visibility into how costs are allocated and adjusted, teams may feel they are not benefiting fairly.
Savings Validation Issues: Ensuring accurate validation of proposed savings can be challenging without robust automated systems.

**Comparison:**

| Team | Initial Savings Proposal | Approved Savings | Final Rate Adjustment |
|------|-------------------------|------------------|-----------------------|
| Dev  | $1,000                   | $350             | -0.02%                |
| QA   | $500                     | $175             | -0.01%                |

**Code:**

```python
# Example IR Model in Python (simplified)
class InternalRateCard:
    def __init__(self):
        self.savings_threshold = 0.15  # Threshold for rate reduction
        self.current_rate = 0.20      # Initial rate

    def adjust_rate(self, total_savings, threshold=None):
        if not threshold:
            threshold = self.savings_threshold
        if total_savings > threshold:
            new_rate = max(0.10, self.current_rate - (total_savings / 100 * 0.05))
            print(f"New rate: {new_rate}%")
            return new_rate
        else:
            print("No adjustment needed.")
            return self.current_rate

# Example Usage
ir = InternalRateCard()
ir.adjust_rate(20)  # Adjusts the rate if savings are significant
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `Initial Rates` (example data) | Adjusted Rate for significant savings (`EC2 instances`: 0.18, `S3 storage`: 0.25, `RDS databases`: 0.15) -> New rate: **0.17%** |

**Data:**

```json
{
    "initial_rates": {
        "EC2 instances": 0.18,
        "S3 storage": 0.25,
        "RDS databases": 0.15
    },
    "savings_threshold": 0.15,
    "current_rate": 0.20
}
```

**Flow:**

1. Initiate Proposal
2. Review and Validate
3. Approve/SuggestModifications
4. Adjust Central IR
5. Redistribute Savings
6. Monitor Performance
7. Revisit and Adjust Periodically

