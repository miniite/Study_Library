---
type: "practice"
difficulty: "Deep"
---

# What is monitors-as-code and how could you manage Datadog monitors with Terraform?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Alert Automation and Runbooks](<../../../../TOOLS/Datadog/Datadog_Platform/Alert_Automation_and_Runbooks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Monitors-as-code (MaaS) involves defining monitoring configurations as code, allowing for version control, automated testing, and consistent deployment of those configurations across environments. Using Terraform to manage Datadog monitors enables a DevOps approach where infrastructure and monitoring are treated equally. This solution ensures that monitoring is not just reactive but proactive, with the added benefits of improved reliability and easier collaboration among team members.

#### Triaging the Question

The question asks about managing Datadog monitors using Terraform within the context of monitors-as-code (MaaS). The core idea here is to shift from static, manually managed configurations to a more dynamic, version-controlled approach. The interviewer is probing your understanding of DevOps practices, infrastructure as code (IaC), and the integration between monitoring tools like Datadog and configuration management tools like Terraform.

From First Principles

Understanding Monitors-as-Code: MaaS means treating monitoring configurations as if they were part of an application's source code. This approach allows teams to manage, version control, and deploy monitoring configurations just like any other piece of infrastructure.

Datadog Monitors: Datadog is a powerful observability platform used for performance monitoring, tracing applications, and ensuring that systems are running smoothly. Monitoring in Datadog involves creating monitors (i.e., checks) to track specific metrics or conditions.

Terraform Integration: Terraform is an infrastructure as code tool that allows teams to define and provision cloud resources using a declarative language. By integrating Terraform with Datadog, you can manage Datadog configurations in the same way as other infrastructure components.

Step-by-Step Solution

Define Monitoring Rules as Code:

Create a directory structure for your Terraform modules where you define each monitor as a separate module or resource.
Each monitor should be defined with its name, type (e.g., metric, log), and the specific conditions it checks.

Terraform Configuration Files:

Use .tf files to define the monitors in Terraform.
HCL
# Example of a simple metric check in Terraform
resource "datadog_monitor" "example" {
  name        = "Example Metric Check"
  type        = "metric alert"
  message     = "The {{ $value }} of {{ $metric }} is {{ $magnitude }}"
  severity    = 3
  threshold_window = 1

  condition {
    comparison                   = "g"
    aggregation_function         = "avg"
    metric                       = "aws.ec2.cpu.utilization{*}"
    threshold                    = 80.0
    time_condition_count         = 5
  }
}

Version Control and Collaboration:

Store these .tf files in a version control system (e.g., Git) to enable collaboration among team members.
Use pull requests to review changes before applying them, ensuring that only validated configurations are deployed.

Automated Testing with Terraform:

Implement unit tests using tools like testlint, which can validate the syntax and logic of your .tf files.
Write integration tests to ensure that your monitors work as expected in a real environment.

Deployment Workflow:

Integrate Terraform into your CI/CD pipeline so that every change to monitoring configurations triggers an automated deployment.
Use state management techniques like remote backends (S3, Consul) to manage shared state across multiple environments (dev, staging, prod).

#### Closing Summary
and Common Mistakes

Managing Datadog monitors with Terraform is a powerful way to leverage the benefits of infrastructure as code for monitoring configurations. This approach ensures that your monitoring setup remains consistent and reliable, while also making it easier for teams to collaborate on critical observability tasks.

#### Common Mistakes

Failing to version control monitor definitions can lead to inconsistency and loss of historical data.
Overlooking automated testing can result in broken or ineffective monitors being deployed.
Not integrating with CI/CD pipelines can make manual deployments error-prone and less efficient.
Final Thoughts

By adopting a DevOps mindset and treating monitoring as code, you can significantly enhance the reliability and maintainability of your observability stack. This approach not only improves operational efficiency but also fosters better collaboration among different teams in your organization.

**Comparison:**

| Monitor Name | Type        | Severity | Metric                     |
|--------------|-------------|----------|----------------------------|
| Example Check | metric alert| 3        | aws.ec2.cpu.utilization{*} |

**Code:**

```hcl
resource "datadog_monitor" "example" {
  name        = "Example Metric Check"
  type        = "metric alert"
  message     = "The {{ $value }} of {{ $metric }} is {{ $magnitude }}"
  severity    = 3
  threshold_window = 1

  condition {
    comparison                   = "g"
    aggregation_function         = "avg"
    metric                       = "aws.ec2.cpu.utilization{*}"
    threshold                    = 80.0
    time_condition_count         = 5
  }
}
```

**Example input / output:**

```text
Input (Terraform configuration):
resource "datadog_monitor" "example" {
  name        = "Example Metric Check"
  type        = "metric alert"
  message     = "The {{ $value }} of {{ $metric }} is {{ $magnitude }}"
  severity    = 3
  threshold_window = 1

  condition {
    comparison                   = "g"
    aggregation_function         = "avg"
    metric                       = "aws.ec2.cpu.utilization{*}"
    threshold                    = 80.0
    time_condition_count         = 5
  }
}

Output (Terraform state):
- name: Example Metric Check
  type: metric alert
  message: The {{ $value }} of {{ $metric }} is {{ $magnitude }}
  severity: 3
  threshold_window: 1
  condition:
    comparison: g
    aggregation_function: avg
    metric: aws.ec2.cpu.utilization{*}
    threshold: 80.0
    time_condition_count: 5
```

**Data:**

```yaml
- name: Example Metric Check
  type: metric alert
  message: The {{ $value }} of {{ $metric }} is {{ $magnitude }}
  severity: 3
  threshold_window: 1
  condition:
    comparison: g
    aggregation_function: avg
    metric: aws.ec2.cpu.utilization{*}
    threshold: 80.0
    time_condition_count: 5
```

**Flow:**

1. Define Monitors as Code
2. Terraform
3. Store in Version Control
4. Automate Testing with Testlint
5. Integrate into CI/CD Pipeline
6. Deploy Consistently AcrossEnvironments

