---
type: "practice"
difficulty: "Deep"
---

# You use Terraform in the deploy pipeline and a plan shows a spurious replace of an RDS instance that would cause an outage; how do you investigate whether it's a provider bug, a moved resource, or a real drift, before anyone hits apply?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Idempotent Deploy Jobs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to thoroughly investigate why Terraform is suggesting a spurious replace of an RDS instance in our CI/CD pipeline. This investigation involves several steps to determine if it’s a provider bug, a moved resource, or actual drift. We'll start by examining the plan output and then proceed with detailed diagnostics.

**Code:**

```hcl
# Example Terraform configuration for an RDS instance
resource "aws_rds_instance" "example" {
  allocated_storage    = 20
  engine               = "aurora"
  engine_version       = "5.7.mysql_aurora.2.09.3"
  instance_class       = "db.t3.medium"
  name                 = "example-instance"
  parameter_group_name = "default.aurora5.7"
}
```

**Example input / output:**

```text
Terraform Plan Output:
---
Resource actions:
  # aws_rds_instance.example will be replaced and then created without changes

Changes to attributes:
  + allocated_storage = 20 -> 30
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Drift Real
6. Not Real
7. Start
8. Is there a provider bug?
9. Check Terraformdocumentation and issuetracker
10. Has the RDS instance movedor been renamed?
11. Review recent operations logsfor any resource movement
12. Verify if there is drift in otherresources related to RDS
13. Report bug and wait for fix
14. Identify the new location ofthe RDS instance
15. Is the drift real or due tomisconfiguration?
16. Fix the misconfigurationcausing drift
17. Apply the plan withoutchanges
18. Retry Terraform plan after bugfix
19. Update resource references incode and re-run plan
20. Document findings for futurereference
21. Repeat steps if issue persists
22. End

