---
type: "practice"
difficulty: "Basic"
---

# What is the difference between deploying a change automatically and requiring a manual approval, for a change that affects alerting?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deployment Observability](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Automatically deploying a change to an alerting system can streamline operations by reducing human intervention, but it carries risks such as potential disruptions if something goes wrong. In contrast, requiring manual approval adds a layer of safety and control, ensuring that changes are thoroughly vetted before deployment. To understand the difference, we need to explore the principles behind each approach.

Firstly, let's define our terms: automatic deployment means the change is pushed without human intervention, while manual approval involves a person reviewing and confirming the change before it goes live.

Now, let's break this down from first principles:

Risk Management:

Automatic Deployment: Speed vs Risk. Faster changes mean quicker updates, but they can also lead to issues if not properly tested.
Manual Approval: Slower but safer. This approach ensures that any potential problems are caught and addressed before the change is deployed.

Operational Efficiency:

Automatic Deployment: Can significantly reduce downtime and improve productivity by automating routine tasks.
Manual Approval: May slow down processes, but it can lead to higher-quality deployments as changes are more thoroughly reviewed.

Recovery and Contingency:

Automatic Deployment: Harder to revert if things go wrong. Automated rollback mechanisms might be necessary.
Manual Approval: Easier to manage rollbacks since the change is under human control, allowing for more granular adjustments.

Cost of Operations:

Automatic Deployment: Lower operational costs due to reduced need for manual oversight but higher development and infrastructure costs.
Manual Approval: Higher operational costs in terms of time spent on reviews, but potentially lower financial and reputational risks.

In practice, a common approach is to use a combination of both methods. For critical changes that affect alerting systems, many organizations opt for a hybrid model where automatic deployment is used after thorough testing, followed by manual approval during production deployments. This way, they get the benefits of speed from automation while maintaining control and safety.

**Code:**

```python
# Example Python code to simulate an automated deployment with alerting changes
def deploy_change(change, environment='production'):
    if environment == 'staging':
        # Perform extensive testing in staging
        test_results = run_staging_tests(change)
        if all(test_results):
            print("Change deployed successfully in staging.")
        else:
            print("Staging tests failed. Deployment aborted.")
    elif environment == 'production':
        # Require manual approval for production changes
        if get_manual_approval():
            apply_production_change(change)
            print("Change applied in production.")
        else:
            print("Manual approval required but not given.")
```

**Example input / output:**

```text
Input (staging environment):
- Change: Update alerting thresholds

Output:
- Staging tests results: [True, True, False]
- Action: Deployment failed due to failing test.

Input (production environment with manual approval):
- Change: Introduce new alerting rules

Output:
- Manual approval prompted.
- User approves the change.
- Production deployment completed successfully.
```

**Data:**

```yaml
# Example YAML configuration for a CI/CD pipeline with manual approval step
stages:
  - name: Staging
    steps:
      - script: echo "Performing staging tests"
      - test_results: [True, True, False]
  - name: Production
    steps:
      - prompt_for_approval:
          question: "Do you approve the change for production?"
      - manual_approver: user123
```

