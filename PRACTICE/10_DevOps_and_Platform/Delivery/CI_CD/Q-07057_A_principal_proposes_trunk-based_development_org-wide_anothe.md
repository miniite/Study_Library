---
type: "practice"
difficulty: "Expert"
---

# A principal proposes trunk-based development org-wide; another insists their regulated team needs release branches for audit. As the deciding architect, how do you reconcile continuous delivery with regulatory change-control?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to first understand the core concepts and trade-offs involved in trunk-based development (TBD) versus release branches. Then, we'll analyze the requirements of both continuous delivery and regulatory compliance, before proposing a balanced solution that leverages the strengths of each approach.

TL;DR: We can reconcile trunk-based development with release branches by adopting a hybrid model where regulated teams use release branches for audit purposes while non-regulated teams continue to benefit from the simplicity and efficiency of trunk-based development. This approach ensures compliance without compromising the agility offered by TBD.

#### Triage
Understanding the Question

The question revolves around reconciling continuous delivery (trunk-based development) with the need for regulatory change-control through release branches. The key is understanding that both approaches serve different purposes, and a balanced hybrid model can address these needs effectively.

#### Assumptions

Regulatory Requirements: The regulated team must maintain strict version control and traceability to meet compliance standards.
Continuous Delivery Benefits: Non-regulated teams need the simplicity and speed of trunk-based development to accelerate their delivery cycles.

#### First-Principles Solution
Step 1: Define Trunk-Based Development (TBD)

Trunk-based development is a continuous integration practice where developers work on features directly in the main branch. This approach ensures that all commits are tested early and often, leading to faster feedback loops and more reliable releases.

Key Benefits of TBD:

Simplified branching and merging.
Continuous testing and quality assurance.
Easier collaboration and merge conflicts resolution.
Step 2: Understand Release Branches

Release branches are specific branches created for preparing a release. They allow developers to make final adjustments, test changes in isolation, and ensure compliance before promoting the code to production.

Key Benefits of Release Branches:

Detailed audit trails.
Isolated testing environment.
Compliance with regulatory standards.
Step 3: Propose a Hybrid Model

Given that different teams have varying needs, we can adopt a hybrid model where:

Regulated Teams: Use release branches for final testing and compliance purposes. This allows them to maintain the necessary audit trails and traceability required by their regulations.
Non-regulated Teams: Continue using trunk-based development for faster iterations and continuous delivery.
Step 4: Implement CI/CD Pipelines

For both types of teams, we can implement a robust CI/CD pipeline that supports both TBD and release branches. This ensures consistency in testing, deployment, and monitoring across the organization.

#### Closing Summary

By adopting a hybrid model that incorporates release branches for regulated teams while maintaining trunk-based development for non-regulated teams, we can effectively reconcile continuous delivery with regulatory compliance. This approach leverages the strengths of both practices to deliver a balanced solution that maximizes agility and ensures compliance.

#### Common Mistakes/Follow-Ups

Ignoring Regulatory Requirements: Failing to account for specific audit trails and traceability needs in regulated environments.
Overcomplicating the Model: Introducing unnecessary complexity by trying to force TBD on all teams, which could lead to slower development cycles and increased merge conflicts.

In conclusion, a balanced hybrid approach is key to successfully implementing CI/CD while meeting regulatory compliance requirements. This model ensures that both regulated and non-regulated teams can operate efficiently without sacrificing the necessary standards.

**Comparison:**

```
Team Type	Development Practice	Key Benefits
Regulated	Release Branches	Audit Trails, Compliance
Non-regulated	Trunk-Based Dev	Simplicity, Speed
```

**Code:**

```python
# Example of a simple CI/CD pipeline configuration
def ci_cd_pipeline():
    # Define stages for TBD and release branches
    stage("Build"):
        script:
            echo "Building code in main branch"

    if release_branch:
        stage("Test Release Branch"):
            script:
                echo "Testing changes in release branch"

        stage("Promote to Prod"):
            script:
                echo "Promoting changes to production with detailed audit trails"
    else:
        stage("Continuous Delivery"):
            script:
                echo "Deploying code directly from main branch"
```

**Example input / output:**

```text
Input:
- Main branch for trunk-based development
- Release branches for regulated teams

Output:
- Continuous integration and testing in main branch
- Final testing and audit trails in release branches before production deployment
```

**Data:**

```json
{
  "main_branch": {
    "name": "trunk",
    "description": "Main development branch for continuous delivery",
    "status": "active"
  },
  "release_branches": [
    {
      "name": "v1.0-hotfix",
      "status": "inactive",
      "last_update": "2023-10-15T14:00Z"
    }
  ]
}
```

**Flow:**

1. Main Branch
2. Continuous Integration
3. Trunk-Based Development(Non-regulated)
4. Release Branches (Regulated)
5. Detailed Testing and AuditTrails
6. Compliance with RegulatoryStandards

