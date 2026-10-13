---
type: "practice"
difficulty: "Deep"
---

# How would you make a CI pipeline detect and block a data pipeline change that would write PII to an unencrypted or wrongly-classified table, using policy-as-code?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Test Data Governance](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Validating_Pipeline_Changes/Test_Data_Governance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure sensitive data like Personally Identifiable Information (PII) is handled securely within our CI/CD pipeline, we need a robust mechanism that can detect and block changes that would write PII to an unencrypted or wrongly classified table. This involves integrating security policies directly into the codebase using policy-as-code, leveraging automated tools for enforcement, and incorporating feedback loops in our CI pipeline.

#### Triage

The core of this question is about implementing a security compliance check within the CI/CD process. The interviewer likely wants to assess your understanding of:

How to define and enforce data classification policies.
Integration with continuous integration tools.
Automating security checks during the build and deployment stages.
Handling policy violations.

#### First Principles Solution

To achieve this, we need to break down the problem into smaller parts:

Define Security Policies: Create a set of rules that determine what constitutes proper data handling practices.
Policy-as-Code Implementation: Use configuration management tools like Terraform or Kubernetes to encode these policies.
Integration with CI/CD Pipeline: Modify the pipeline steps to include security checks before deployment.
Feedback Loops and Notifications: Set up mechanisms for alerting stakeholders when policies are violated.
Step-by-Step Solution
1. Define Security Policies

We start by defining what constitutes a correctly classified and encrypted table. For instance, we might create a policy that states:

PII should only be stored in tables marked with the PII label.
Encryption keys must be managed using a secure key management service.
2. Policy-as-Code Implementation

We can use tools like AWS Config or Open Policy Agent (OPA) to define these policies as code. Here's an example of an OPA policy snippet:

REGO
package data_security

pii_table = input.resource.label.PII == true and input.resource.table_encryption == "enabled"
violation := not pii_table
violation

This Rego policy checks that only tables with the PII label and enabled encryption are used for storing PII.

3. Integration with CI/CD Pipeline

Next, we integrate these policies into our CI/CD pipeline. For example, using Jenkins or GitLab CI, we can add a step to validate code changes against the defined security policies:

YAML
stages:
  - name: Validate
    jobs:
      - job: validate-security-policies
        config:
          image: opa:latest
          commands:
            - opa eval --input ./policies.rego --data ./config.json > result.json
            - if ! cat result.json | jq -e '.violation'; then echo "Security policy violation detected"; exit 1; fi

pipelines:
  branches:
    master:
      - step: validate-security-policies

This Jenkinsfile snippet uses OPA to evaluate the policies defined in policies.rego against the current state of our infrastructure, stored in config.json.

4. Feedback Loops and Notifications

Finally, we need mechanisms to notify stakeholders when security policies are violated:

Email Alerts: Set up automated emails to alert developers or security teams.
Dashboard Updates: Integrate with monitoring tools like Prometheus or Grafana to update dashboards.
Continuous Monitoring: Implement real-time monitoring using tools like AWS CloudWatch or Datadog.

#### Closing Summary

In summary, we have integrated a policy-as-code approach into our CI/CD pipeline to ensure that any changes writing PII to an unencrypted table are detected and blocked. This involves defining clear security policies, implementing these policies as code with OPA, integrating this into the CI/CD steps, and setting up robust feedback loops.

#### Common Mistakes/Follow-Ups
Not Defining Clear Policies: Failing to clearly define what constitutes a violation can lead to ambiguous enforcement.
Missing Feedback Mechanisms: Without proper notifications, violations might go unnoticed until it's too late.
Overly Complex Policy Rules: Overly complex policies can make the system difficult to manage and understand.

For follow-ups, one could explore how different cloud providers (like AWS or GCP) offer similar capabilities through their native services. Additionally, discussing trade-offs between strict enforcement versus flexibility in policy definitions might be useful.

**Comparison:**

```
Policy Rule	Description
PII Label	Ensures that only tables labeled with PII can store sensitive data.
Table Encryption	Requires that all tables storing PII are encrypted.
Step in CI Pipeline	Action
Validate	Uses OPA to evaluate policies and block violations.
Notify	Sends alerts via email or dashboard updates.
```

**Code:**

```

stages:

name: Validate jobs:
job: validate-security-policies config: image: opa:latest commands: - opa eval --input ./policies.rego --data ./config.json > result.json - if ! cat result.json | jq -e '.violation'; then echo "Security policy violation detected"; exit 1; fi

pipelines: branches: master: - step: validate-security-policies

CODE
Copy

```rego
package data_security

pii_table = input.resource.label.PII == true and input.resource.table_encryption == "enabled"
violation := not pii_table
violation
```

**Example input / output:**

```text
# Running OPA evaluation on a sample configuration
$ opa eval --input ./policies.rego --data ./config.json > result.json

# Result of the evaluation
$ cat result.json
{
  "violation": false
}

$ if ! cat result.json | jq -e '.violation'; then echo "Security policy violation detected"; fi
# Output: (nothing, indicating no violation)
```

**Data:**

```json
{
  "resource": {
    "label": {"PII": true},
    "table_encryption": "enabled"
  }
}
```

**Flow:**

1. Yes
2. No
3. CI Pipeline Start
4. Code Change
5. Validate Security Policies
6. Policy Violation?
7. Block Deployment
8. Continue Pipeline
9. Deploy Changes

