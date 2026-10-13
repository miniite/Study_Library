---
type: "practice"
difficulty: "Deep"
---

# How does Pulumi's Python SDK differ from writing a boto3 provisioning script, and when would you pick imperative Python provisioning over declarative Terraform for your team?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Safe User-authored Configuration](<../../../../TOOLS/Python/Python_Language/Safe_User-authored_Configuration/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, let's first summarize the key differences between Pulumi's Python SDK and using boto3 scripts for AWS resource provisioning. Then, we'll delve into when to choose imperative Python provisioning over declarative Terraform.

TL;DR: Pulumi’s Python SDK allows you to use familiar Python constructs for infrastructure as code (IaC), whereas boto3 is a lower-level library for interacting with AWS services directly. Pulumi is typically chosen for its declarative approach, which aligns well with modern IaC practices and tooling. However, in certain scenarios, imperative Python provisioning might be more suitable.

#### Triage
The core of this question involves understanding the trade-offs between declarative (like Terraform) and imperative programming models when using Python for AWS infrastructure provisioning. Pulumi’s Python SDK leverages Python's syntax and libraries like boto3 to provide a declarative model that integrates seamlessly with existing CI/CD pipelines and tooling.

Key Assumptions:

You are familiar with both Pulumi and Terraform.
Your team has experience in using Python for scripting and automation.
The goal is to compare these tools based on their use cases, performance, and developer productivity.

#### First-Principles Solution

Pulumi's Python SDK vs boto3:

Pulumi: Offers a high-level API that abstracts away much of the boilerplate associated with resource management and deployment. It uses YAML or JSON templates for a declarative definition of infrastructure, making it easier to version control changes.
boto3: A lower-level library that provides direct interaction with AWS services using Python’s syntax. It is more verbose and requires explicit code to manage resources.

Declarative vs Imperative:

Declarative (Pulumi): Focuses on describing what the desired state of your infrastructure should be, not how to achieve it. This model is easier to reason about and maintain.
Imperative (boto3): Describes a series of steps to achieve a specific outcome. This approach can be more flexible but harder to debug and version control.

Use Cases:

Pulumi: Ideal for teams with existing Python expertise who need to integrate infrastructure provisioning directly into their codebase. It integrates well with modern CI/CD tools like GitHub Actions, Jenkins, etc.
boto3: Suitable when you have specific use cases that require fine-grained control over AWS services or when integrating with legacy systems.

Performance and Scalability:

Both Pulumi and boto3 can be optimized for performance. However, Pulumi’s high-level abstractions might offer better scalability due to its focus on state management and resource orchestration.

Developer Productivity:

Declarative models like Pulumi generally lead to higher developer productivity because they are closer to the language developers are familiar with and require fewer lines of code for complex infrastructure setups.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, while Pulumi's Python SDK provides a more integrated and declarative approach to provisioning AWS resources, there may be situations where imperative Python provisioning via boto3 is more appropriate. For example, if your team has specific requirements that necessitate fine-grained control over the provisioning process, or if you're working on legacy systems with existing Python codebases.

#### Common mistakes
include:

Overlooking the nuances between declarative and imperative approaches.
Failing to consider the integration challenges when moving from a boto3-based solution to Pulumi’s higher-level constructs.
Not leveraging the extensive community support and tooling available for Pulumi, which can significantly enhance developer productivity.

When considering which approach to use, always evaluate based on your team's skills, project requirements, and long-term maintainability goals. For teams with existing Python expertise, Pulumi may offer a more streamlined path to infrastructure as code.

**Comparison:**

```
Tool	Model	Key Features
Pulumi (Python SDK)	Declarative	High-level abstractions, integrated with CI/CD tools
boto3	Imperative	Lower-level library, direct AWS service interaction
```

**Code:**

```python
# Example of using boto3 for provisioning an S3 bucket (imperative)
import boto3

s3 = boto3.resource('s3')
bucket_name = 'my-new-bucket'

try:
    s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={'LocationConstraint': 'us-east-1'})
except Exception as e:
    print(f"Error creating bucket: {e}")

# Example of using Pulumi's Python SDK for the same (declarative)
import pulumi
from pulumi_aws import s3

bucket = s3.Bucket("my-new-bucket")
```

