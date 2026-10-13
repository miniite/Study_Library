---
type: "practice"
difficulty: "Intermediate"
---

# Why do teams avoid letting people edit alert rules directly in the UI when the rules are supposed to come from git?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rollback Strategy](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, teams often prefer managing alert rules via code and Git over editing them directly in a user interface (UI). This approach aligns with broader DevOps practices that emphasize automation, version control, and collaboration through shared repositories. The reasoning behind this preference involves several key aspects such as consistency, traceability, and security.

The question is probing into the rationale for using CI/CD practices to manage alert rules rather than allowing direct UI edits. Key assumptions include:

The team values a robust change management process.
There's an emphasis on maintaining historical context of changes.
Security concerns are paramount in terms of who can make changes and when.

Let’s break this down from first principles:

#### Cons
istency: When alert rules are managed via code, they benefit from the same consistency checks that any other code does. This ensures that rules adhere to expected formats and behaviors. Direct UI edits might introduce inconsistencies if not handled carefully.

Traceability & History: Version control systems provide a rich history of changes made over time. This makes it easier to understand who changed what, when, and why. In contrast, direct UI edits often lack this audit trail, making it harder to track the evolution of alert rules.

Security: By managing alert rules through Git, teams can enforce access controls more effectively. Only developers with specific permissions can push changes, reducing the risk of accidental or malicious modifications. Direct UI edits might be less restrictive and could lead to security breaches if not properly secured.

Testing & Validation: Changes made via code can go through a testing pipeline before being deployed. This ensures that any changes do not disrupt the system in unexpected ways. Direct UI edits, on the other hand, bypass this crucial validation step.

Collaboration & Code Reviews: When alert rules are part of a shared repository, they can be reviewed by peers. This collaborative process helps catch issues early and ensures that the best practices are followed consistently.

In summary, while direct UI editing might seem more convenient at first glance, it lacks many of the critical benefits provided by CI/CD pipelines. Teams avoid this approach to ensure their systems remain robust, secure, and maintainable over time.

**Code:**

```yaml
# Example alert rule in YAML format (for a hypothetical AIOps system)
apiVersion: monitoring.example.com/v1
kind: AlertRule
metadata:
  name: memory_usage_alert
spec:
  conditions:
  - conditionType: ThresholdCrossing
    threshold:
      value: 80
      comparator: '>'
    targetMetric: memory_usage_percentage
  actions:
  - actionType: Notify
    targets:
    - alert_email@example.com
```

**Example input / output:**

```text
Dry run example:

**Input**: A developer makes a change to an alert rule using Git.

1. Developer commits changes with a meaningful message and pushes to the repository.
2. CI/CD pipeline runs tests on the new commit.
3. If successful, the new rule is deployed in the monitoring system.
4. Alerting behavior changes according to the updated rules.
```

**Flow:**

1. Yes
2. No
3. Developer Commits Changes
4. Run Tests?
5. Deploy Rules
6. Fix and Retry
7. Monitor System Behavior

