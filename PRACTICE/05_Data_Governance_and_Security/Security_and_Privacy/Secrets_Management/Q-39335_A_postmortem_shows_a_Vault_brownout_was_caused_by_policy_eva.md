---
type: "practice"
difficulty: "Deep"
---

# A postmortem shows a Vault brownout was caused by policy evaluation getting expensive as glob-heavy policies grew. Explain how policy structure affects evaluation cost at scale and how you'd refactor to keep it fast.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Vault Outage Resilience](<../../../../05_Data_Governance_and_Security/Security_and_Privacy/Secrets_Management/Vault_Outage_Resilience.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Vault's policy evaluation is influenced significantly by its structure, especially when using glob patterns in policies. To mitigate performance issues caused by complex glob-heavy policies, we need to refactor these policies to use more efficient matching techniques and consider a hierarchical or compartmentalized approach.

#### Triaging the Question

The question revolves around understanding how Vault's policy evaluation cost scales with complex glob-heavy policies and suggests refactoring strategies to maintain fast evaluations. The key points are:

Complexity of Glob Patterns: How using glob patterns can lead to exponential growth in evaluation time.
Evaluation Cost: The computational overhead associated with evaluating these policies, which can become a bottleneck as the number of policies and their complexity increase.
Refactoring Strategies: Techniques to optimize policy structure for better performance.

#### Assumptions
We are dealing with Vault version 1.x or later, which supports glob patterns.
The existing policies are complex and use extensive globbing in multiple layers.
The system has a large number of policies and secrets, leading to high evaluation costs during operations like read/write.

#### First-Principles Solution
1. Understanding Glob Patterns:

Glob patterns (e.g., secrets/*/data/*) match files based on wildcard characters. These patterns can be expensive because the engine needs to recursively search through all directories and secrets, leading to a high number of evaluations and potential performance bottlenecks.

2. Evaluation Cost Analysis:
Linear Complexity: Simple glob patterns like secrets/data/* may evaluate linearly with the number of secrets.
Exponential Complexity: Complex patterns involving multiple layers (e.g., secrets/*/data/*/value) can lead to exponential evaluation times as Vault needs to traverse each level recursively.
3. Refactoring Strategies:

To mitigate these issues, we need to adopt more efficient policy structures:

Hierarchical Policies:
Organize policies in a hierarchical manner, similar to how directory structures are organized.
Use path-based policies that avoid deep nesting:
HCL
# Example of a hierarchical policy structure
path "secrets/data/*" {
  capabilities = ["read", "list"]
}
Role-Based Policies:
Define roles based on job functions or operational needs.
Assign permissions to these roles rather than individual secrets, which can reduce the number of policies and evaluation complexity.
Least Privilege Principle:
Ensure that each policy grants only necessary privileges.
Use of default Policy:
Create a default policy that applies to all unmatched paths, reducing the need for exhaustive matching.
4. Implementation Example:
HCL
# Default policy for unknown paths
path "secrets/data/*" {
  capabilities = ["read", "list"]
}

# Role-based policies
role "db-admin" {
  path "secrets/db/*" {
    capabilities = ["read", "update"]
  }
}

role "app-read-only" {
  path "secrets/app/*" {
    capabilities = ["read"]
  }
}
5. Performance Testing and Monitoring:
Implement performance monitoring to track policy evaluation times.
Use Vault's built-in metrics to identify bottlenecks and areas for optimization.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

Vault's policy evaluation is highly dependent on the structure of its policies, particularly when using complex glob patterns. By adopting a hierarchical and role-based approach, we can significantly reduce evaluation costs and ensure more efficient operations. Common mistakes include overusing deep globbing or neglecting least privilege principles.

For follow-ups, consider:

Scaling Considerations: How these strategies scale with increasing numbers of policies and secrets.
Security Implications: Ensuring that the refactored policies maintain security without introducing vulnerabilities.
Performance Tuning: Fine-tuning policy evaluation through Vault's configuration options.

