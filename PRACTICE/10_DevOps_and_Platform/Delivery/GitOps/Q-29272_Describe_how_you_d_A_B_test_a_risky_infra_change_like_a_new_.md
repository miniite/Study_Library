---
type: "practice"
difficulty: "Deep"
---

# Describe how you'd A/B test a risky infra change (like a new ingress controller) using GitOps across a canary cluster before fleet-wide rollout.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To A/B test a risky infrastructure change like deploying a new ingress controller, leveraging GitOps principles ensures a controlled and safe deployment strategy. The core idea is to use a staging or canary cluster where we apply changes in a safe environment before they are applied fleet-wide. Here’s how you would approach this:

#### Overview
This process involves setting up a separate cluster (canary) that mirrors the production environment, deploying the new ingress controller there first. Once validated and deemed stable, it is rolled out to the main production clusters.

#### Triage
The question is about ensuring robust A/B testing for risky infrastructure changes using GitOps. We need to set up a staging cluster, apply changes via GitOps workflows, validate them, and then perform a controlled rollout.

#### First-Principles Solution

Setup Staging Cluster: Create a canary cluster that is an exact replica of the production environment.
GitOps Workflow: Use a GitOps tool (like Flux) to manage infrastructure as code in repositories, ensuring both clusters are updated from the same source of truth.
Rollout Strategy: Implement a gradual rollout strategy where traffic is split between the old and new ingress controllers.
Monitoring & Validation: Continuously monitor the canary cluster for any issues. Once validated, perform an A/B test to ensure it works as expected before fully rolling out.

#### Closing Summary
+ Common Mistakes:

Summary: By using a canary cluster with GitOps, we ensure that risky changes are thoroughly tested in a controlled environment before they impact the main production clusters.

#### Common Mistakes
One common mistake is skipping thorough validation in the staging cluster or performing an all-or-nothing rollout without sufficient monitoring. Another issue could be not having proper fail-safes or rollback mechanisms in place.

**Comparison:**

```
Cluster	Ingress Controller Version
Production	Old
Canary	New
```

**Code:**

```yaml
# Example YAML for creating a canary cluster using Terraform
resource "aws_launch_template" "canary_cluster" {
  name_prefix = "canary-cluster"
  image_id     = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

resource "aws_autoscaling_group" "canary_asg" {
  launch_template {
    id            = aws_launch_template.canary_cluster.id
    version       = "$Latest"
  }

  min_size = 1
  max_size = 3

  vpc_zone_identifier = [aws_subnet.public_subnets[0].id]
}
```

**Example input / output:**

```text
Input:
- Initial setup of the canary cluster with Flux and GitOps deployment.
- Deployment of new ingress controller in canary cluster.

Output:
- Canary cluster is stable and validated without issues.
- Traffic split between old (90%) and new (10%) ingress controllers.
- Once validated, full rollout to production clusters.
```

**Data:**

```json
{
  "clusters": {
    "production": {
      "replicas": 2,
      "ingress_controller": "old"
    },
    "canary": {
      "replicas": 1,
      "ingress_controller": "new"
    }
  },
  "traffic_split": {
    "production": 90,
    "canary": 10
  }
}
```

**Flow:**

1. Setup Canary Cluster
2. Initialize Flux
3. Deploy New Ingress Controllerin Canary
4. Mirrored Deployment toProduction
5. Traffic Split 90% Old, 10% New
6. Monitor Stability andPerformance
7. Rollout to Full Cluster ifSuccessful

