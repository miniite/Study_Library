---
type: "practice"
difficulty: "Deep"
---

# Two teams' pipelines both apply Terraform to overlapping infrastructure and keep fighting over the same resources; how would you carve state and ownership so their deploys stop stepping on each other?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Shared Pipeline Changes](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a scenario where two teams are deploying into shared infrastructure using Terraform, and their pipelines frequently conflict, the primary challenge is to ensure that both teams can operate independently without interfering with each other's state files or deployments. This problem requires careful planning of state management and resource ownership to avoid conflicts.

#### Triaging the Question

The question essentially asks how we can design a CI/CD infrastructure that allows two independent development teams to manage overlapping Terraform configurations while preventing their pipelines from stepping on each other’s toes. The key issues are:

State Management: Ensuring both teams have access to their own state files without conflicting.
Ownership and Versioning: Defining clear ownership of resources and managing versioning to avoid deployment race conditions.
Parallel Execution: Allowing concurrent but non-overlapping execution of pipelines.

#### First-Principles Solution

To address the problem, we can implement a solution that separates state management and resource ownership between teams while ensuring both can deploy independently. Here’s a step-by-step approach:

Centralized State Storage with Separate Buckets: Use a centralized backend for Terraform states but assign separate S3 buckets or DynamoDB tables for each team to store their own state files.
Versioned States and Locking Mechanism: Implement versioning of the state files to track changes over time and ensure that each deployment updates the correct version. Additionally, use locking mechanisms to prevent concurrent writes from conflicting teams.
Resource Ownership and Tagging: Tag resources with ownership metadata (e.g., team_name) so that Terraform knows which team is responsible for specific resources. This also helps in auditing and managing permissions.
Customized Workflows and Pipelines: Tailor the CI/CD pipelines to handle state updates and deployments based on the versioned states, ensuring that each deployment operates independently.
Detailed Implementation

State Management

Use an S3 bucket per team for storing Terraform states.

#### Example
team-a-state and team-b-state.

Versioning States with DynamoDB

Store state versions in a DynamoDB table, where the primary key is a combination of state_bucket and version.

#### Example
schema:
JSON
{
  "StateBucket": "team-a-state",
  "Version": 1,
  "StateData": "...", // Encoded Terraform state data
  "LastModified": ISO_TIMESTAMP
}

Locking Mechanism

Implement a locking mechanism using AWS Lambda or other infrastructure to prevent multiple pipelines from writing to the same state simultaneously.

#### Example
Use an SNS topic with a dedicated subscriber that waits for lock release before performing state updates.

Resource Ownership and Tagging

Tag resources in Terraform with team_name metadata.
HCL
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    team_name = "team-a"
  }
}
Use these tags to ensure that only the correct pipeline can modify resources owned by a specific team.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, we need to implement state management strategies that separate each team’s Terraform states, versioning mechanisms for tracking changes, locking mechanisms to prevent race conditions, and tagging to enforce resource ownership. By doing so, both teams can operate independently without stepping on each other's toes.

#### Common mistakes
include not implementing proper versioning or locking, leading to data corruption or race conditions. Another pitfall is failing to properly tag resources, which could result in accidental modifications by the wrong team.

#### Follow-Ups
How would you ensure security and access controls for these state buckets?
What if one team needs to use resources from another team? How do you handle that scenario?
In a highly dynamic environment with frequent changes, how often should states be versioned and managed?

**Comparison:**

| Resource | Team Name |
|----------|-----------|
| aws_instance.example | team-a    |
| aws_instance.example2 | team-b   |

**Code:**

```python
# Example of tagging in Terraform
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    team_name = "team-a"
  }
}
```

**Example input / output:**

```text
Input:
- Team A has a Terraform configuration to update its instances.
- Team B has a similar configuration for its own resources.

Output:
- Both teams run their pipelines independently without interfering with each other's state files or deployments.
- State updates are versioned and locked to prevent conflicts.
```

**Data:**

```json
{
  "StateBucket": "team-a-state",
  "Version": 1,
  "StateData": "...", // Encoded Terraform state data
  "LastModified": "2023-10-01T12:00:00Z"
}
```

**Flow:**

1. Team A Pipeline
2. Apply Lock?
3. Update State Version
4. Write to S3 Bucket
5. Wait for Lock Release
6. Team B Pipeline
7. Apply Lock?
8. Update State Version
9. Write to S3 Bucket
10. Wait for Lock Release
11. Unlock
12. Unlock

