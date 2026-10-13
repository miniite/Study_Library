---
type: "practice"
difficulty: "Intermediate"
---

# Why did you choose a managed service over self-hosting for your last project?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Managed Services vs Self-hosting](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Managed_Services_vs_Self-hosting/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between a managed service and self-hosting, the primary factors to consider include cost, scalability, maintenance, security, and compliance. For my last project, I chose a managed service over self-hosting due to several key advantages that aligned with our project's requirements.

Firstly, managed services often come with built-in high availability, backup strategies, and disaster recovery options, which can significantly reduce the risk of downtime and data loss. Additionally, these services are typically operated by experts who handle security patches, software updates, and system monitoring around-the-clock. This reduces the burden on our development team, allowing us to focus more on core business logic rather than infrastructure management.

Secondly, managed services often offer better scalability options. For instance, cloud providers can quickly provision additional resources as needed without requiring changes in our codebase or infrastructure configuration. This flexibility is crucial for projects that might experience unpredictable spikes in traffic.

Lastly, the cost of self-hosting can be substantial when accounting for hardware costs, energy consumption, and dedicated personnel to manage the environment. Managed services often provide predictable pricing models with no upfront capital expenditure, making them more cost-effective in the long run.

**Code:**

```text
# Example of provisioning a managed service
# This is pseudo-code representing the process

managed_service_provisioning = {
  "service_name": "AWS Elastic Kubernetes Service (EKS)",
  "regions_available": ["us-east-1", "eu-west-2"],
  "auto_scaling": True,
  "security_group": "sg-0123456789abcdef0",
  "node_type": "m5.large"
}
```

