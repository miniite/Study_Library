---
type: "practice"
difficulty: "Deep"
---

# How would you test your Vault disaster recovery runbook so you're confident it works before a real outage?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vault Outage Resilience](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Outage_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure confidence in our Vault disaster recovery runbook, we need to perform thorough testing that mimics a real-world scenario as closely as possible. This involves several steps, including setting up test environments, creating realistic failure scenarios, and verifying the recovery process. By doing this, we can identify potential issues before they become critical during an actual outage.

First, let's break down what the question is asking: The interviewer wants to know how you would validate that your Vault disaster recovery runbook functions correctly in a worst-case scenario without causing downtime or data loss in the production environment. This includes testing both the failover process and the subsequent recovery steps.

We'll start by making some key assumptions:

We have multiple Vault instances, including one primary and potentially several secondary replicas.
The failover process involves switching from the primary to a secondary instance if the primary goes down.
We need to verify that all critical data remains accessible post-failover.
Our recovery process includes restoring the primary instance back after resolving issues.

Now, let's reason step by step from first principles:

Setup Test Environments:

Create an isolated test environment that mirrors our production setup.
Configure Vault instances with similar settings as in production, including storage backends and access controls.

Simulate a Failure Scenario:

Simulate the failure of the primary Vault instance using tools like Kubernetes or on-premises infrastructure management.
This could involve stopping the service, causing network issues, or introducing data corruption through controlled means.

Run the Failover Process:

Trigger the failover process within your runbook to promote a secondary instance as the new primary.
Ensure that all clients can still read and write secrets from this new primary instance seamlessly.

Verify Data Consistency and Accessibility:

Use a tool like Vault's built-in consistency checks or third-party tools (e.g., k8s Vault Operator) to verify data integrity across both instances.
Manually check key operations such as secret retrieval, write operations, and versioning.

Test the Recovery Process:

Once you're confident in the failover process, simulate a recovery scenario where the original primary instance is brought back online after addressing any issues.
Verify that the original primary can resume its role without disrupting the current primary or secondary instances.
Ensure data consistency and proper state synchronization between both instances.

Document and Automate:

Document the test results, including any identified issues and their resolutions.

#### Cons
ider automating parts of this process to make future tests more efficient and reliable.

Review and Update the Runbook:

Based on testing outcomes, update your disaster recovery runbook if necessary.
Conduct a code review or peer review of the updated runbook to ensure best practices are followed.

#### Closing Summary
By systematically testing our Vault disaster recovery process in a controlled environment, we can build confidence that it will work when needed. This approach involves setting up realistic test scenarios, validating failover and recovery processes, ensuring data consistency, and documenting results for continuous improvement.

#### Common Mistakes/Follow-Ups

Failing to simulate all possible failure modes (e.g., partial outages vs. complete loss of service).
Not verifying data integrity and access control post-failover.
Underestimating the time and effort required for full test coverage, leading to incomplete or superficial testing.

**Comparison:**

| Step | Action | Output |
|------|--------|--------|
| 1    | Stop primary instance | Vault service stopped on 'vault-primary' container. |
| 2    | Promote secondary to primary | Vault service promoted from 'secondary-01' to 'primary-01'. |
| 3    | Verify access to a secret | {"request_id": "3e7d64c2-fb8a-6c9f-cbe0-d5856f158e8d", ...} |

**Code:**

```bash
# Example Shell Script for Simulating Failover Process

# Step 1: Stop Primary Instance
docker-compose stop vault-primary

# Step 2: Promote Secondary to Primary
./promote-secondary-to-primary.sh secondary-01

# Step 3: Verify Access
vault read secret/my-secret

# Step 4: Bring Back the Original Primary
./bring-back-original-primary.sh primary-01

# Step 5: Validate Consistency
docker exec -it vault-primary vault consistency check --format json > /tmp/check.json
```

**Example input / output:**

```text
Input:
- Stop primary instance

Output:
Vault service stopped on 'vault-primary' container.

Input:
- Promote secondary to primary

Output:
Vault service promoted from 'secondary-01' to 'primary-01'.

Input:
- Verify access to a secret

Output:
{
  "request_id": "3e7d64c2-fb8a-6c9f-cbe0-d5856f158e8d",
  "lease_id": "",
  "renewable": false,
  "lease_duration": -1,
  "data": {
    "data": {
      "value": "password123"
    },
    "metadata": {}
  }
}
```

**Data:**

```json
{
  "primary": {
    "name": "vault-primary",
    "status": "active"
  },
  "secondary": [
    {
      "name": "secondary-01",
      "status": "passive"
    }
  ]
}
```

**Flow:**

1. Stop Primary Instance
2. Promote Secondary to Primary
3. Verify Access
4. Bring Back Original Primary
5. Validate Consistency
6. Document and UpdateRunbook

