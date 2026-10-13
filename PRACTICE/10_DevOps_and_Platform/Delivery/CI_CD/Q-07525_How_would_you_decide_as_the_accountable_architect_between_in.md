---
type: "practice"
difficulty: "Deep"
---

# How would you decide, as the accountable architect, between investing in ephemeral per-PR full-stack environments versus a shared high-fidelity staging environment for a 4000-engineer org, quantifying cost, data-seeding fidelity, and the contention/flakiness the shared model causes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Ephemeral Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Ephemeral_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between ephemeral per-PR (Pull Request) full-stack environments versus a shared high-fidelity staging environment in a large 4000-engineer organization, as an architect, we must balance cost efficiency, data fidelity, and the operational overhead. The key is to weigh these factors against the potential disruptions caused by contention and flakiness.

The core of this decision involves understanding that ephemeral environments offer more granular control over testing conditions but come with higher resource costs due to frequent environment creation and teardown cycles. Conversely, a shared high-fidelity staging environment offers economies of scale in terms of infrastructure, yet introduces challenges such as data consistency across multiple concurrent PRs and potential bottlenecks from contention.

To fully address this question, we'll start by quantifying the cost implications, then assess the fidelity of data seeding, and finally evaluate the operational issues related to shared environments. Let's break it down step by step:

Cost Analysis:

Ephemeral Environments: These are created and destroyed for each PR. Assuming an average environment life cycle of 4 hours per PR (a generous estimate), we can calculate the total number of environments needed over a month to determine the cost.
Shared Environment: This approach requires fewer resources but may need more robust infrastructure to handle concurrent usage, such as load balancers and scaling strategies.

Data Fidelity:

Ephemeral Environments: Typically allow for complete isolation, which can be beneficial in maintaining data integrity during testing.
Shared Environment: May require complex mechanisms like database cloning or snapshotting to ensure that each PR tests against a consistent state of the data.

Operational Overhead and Contention:

Ephemeral Environments: Frequent creation and teardown cycles can lead to increased operational overhead, including time spent setting up environments.
Shared Environment: Higher contention for shared resources can introduce flakiness in tests and delays in PR reviews due to longer wait times.

Given these factors, we need a detailed analysis:

Cost Calculation:

Assume an average environment cost of $0.50 per hour, with each PR taking 4 hours.
Monthly PRs: A rough estimate based on development velocity might be 120 PRs/month (assuming 4 engineers per team and 30 teams).
Cost for Ephemeral Environments: (120 \text{ PRs} \times 4 \text{ hours/PR} \times $0.50/\text{hour} = $240)
Shared Environment Cost: More complex but potentially lower due to shared infrastructure, say $300/month for equivalent capacity.

Data Fidelity Assessment:

Ephemeral Environments: Each PR environment can be seeded with a clean copy of the production database, ensuring data consistency.
Shared Environment: Requires a robust data management strategy, such as regular snapshots or cloning mechanisms. This could add complexity and cost.

Operational Impact:

Ephemeral Environments: Higher operational overhead but less contention.
Shared Environment: Lower operational overhead due to shared resources but higher risk of flakiness from contention.

In summary, the decision hinges on balancing these trade-offs. For a large organization like ours, the cost savings and reduced operational complexity of a shared high-fidelity staging environment might outweigh the increased risk of data inconsistency and operational bottlenecks.

#### Common mistakes
include overlooking the true cost implications or underestimating the impact of contention. It's crucial to involve both engineering teams and finance to get accurate cost estimates and to conduct thorough stress testing of any chosen model.

**Comparison:**

| PRs Created | Ephemeral Cost (USD) | Shared Cost (USD) |
|-------------|---------------------|------------------|
| 25          | $50                 | $12.5            |
| 60          | $120                | $30              |
| 90          | $180                | $45              |

**Code:**

```python
# Example Cost Calculation for Ephemeral Environments
num_pRs_per_month = 120
hours_per_PR = 4
cost_per_hour = 0.50

total_cost_ephemeral = num_pRs_per_month * hours_per_PR * cost_per_hour
print(f"Total cost for ephemeral environments: ${total_cost_ephemeral}")
```

**Example input / output:**

| PRs Created | Ephemeral Cost (USD) | Shared Cost (USD) |
|-------------|---------------------|------------------|
| 25          | $50                 | $12.5            |
| 60          | $120                | $30              |
| 90          | $180                | $45              |

**Data:**

```yaml
# Example Data Model for Shared Environment
staging_environment:
  - database_name: prod_snapshot_20231015
  - environment_type: shared
  - max_concurrent_pRs: 100
  - cost_per_month: 300
```

**Flow:**

1. Yes
2. No
3. PR Creation
4. Is Ephemeral?
5. Ephemeral Environment Setup
6. Shared Environment Access
7. Environment Teardown
8. Run Tests

