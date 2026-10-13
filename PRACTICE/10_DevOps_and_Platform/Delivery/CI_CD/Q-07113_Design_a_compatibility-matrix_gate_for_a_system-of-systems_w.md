---
type: "practice"
difficulty: "Expert"
---

# Design a compatibility-matrix gate for a system-of-systems where any two services deployed at incompatible version pairs cause outages; how do you compute allowed version combinations, enforce them at deploy time, and keep the matrix from becoming a deploy-blocking bottleneck?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Deployment Gating Trade-offs](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Shared_Pipeline_Changes/Deployment_Gating_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a compatibility-matrix gate for a system-of-systems where incompatible service versions cause outages, we need to ensure that only version combinations deemed compatible by our defined rules can be deployed. This involves computing allowed version combinations, enforcing them at deploy time, and managing the matrix efficiently without becoming a bottleneck.

First, let's break down the problem:

Compute Allowed Version Combinations: We need an algorithm that determines which service versions are compatible with each other.
Enforce at Deploy Time: Implement mechanisms to check compatibility before allowing deployments.
Matrix Management: Ensure the system scales well and doesn't become a performance bottleneck.

#### Triaging the Question

The interviewer is probing your understanding of CI/CD pipelines, versioning strategies, and how to handle complex dependencies in a multi-service environment. Key assumptions include:

Services can be versioned with semantic versioning (e.g., 1.2.3).
Incompatibilities are defined by business rules or service contracts.
Deployments are automated through CI/CD pipelines.
Step-by-Step Solution

Define Compatibility Rules: Establish clear, documented rules for what constitutes a compatible version pair. For example:

Major versions should be backward-compatible.
Minor versions can introduce new features but must not break existing functionality.
Patch versions fix bugs without changing APIs.

Compute Allowed Version Combinations:

Use graph theory to model service dependencies as nodes and incompatibilities as edges.
Implement a dynamic programming approach or use an algorithm like Dijkstra's for finding all valid paths (version combinations) between services.

Enforce at Deploy Time:

Integrate the compatibility check into your CI/CD pipeline.
Use tools like Jenkins, GitLab CI, or Kubernetes to trigger these checks automatically before a deployment.

Efficient Matrix Management:

Store the compatibility matrix in a database (e.g., PostgreSQL) for quick lookups.
Cache frequently used queries to reduce load on the database.
Optionally, use a distributed cache like Redis to speed up real-time checks during deployments.

#### Closing Summary
and Common Mistakes/Follow-Ups

By carefully defining and managing compatibility rules, we can prevent service outages due to version incompatibilities. Key challenges include ensuring scalability of the matrix check process and avoiding overly complex matrix computations that could become bottlenecks. Common mistakes might involve not thoroughly documenting compatibility rules or ignoring edge cases like deprecated versions.

#### Follow-ups
could explore specific implementation details, such as:

How to handle rapid changes in service versions?
What if some services have non-standard versioning schemes?

**Comparison:**

```
Service A	Compatible Versions
1.2.3	2.0.1, 2.1.0
2.0.1	1.2.3, 2.1.0
```

**Code:**

```python
# Example of a simple compatibility check function
def is_compatible(service_a_version, service_b_version):
    # Example logic based on semantic versioning rules
    def parse_version(version_str):
        parts = version_str.split('.')
        return tuple(map(int, parts))

    v_a, v_b = parse_version(service_a_version), parse_version(service_b_version)
    if v_a[0] != v_b[0]:  # Major versions must match
        return False
    if v_a[1] > v_b[1]:  # Minor version of A should not be higher than B
        return False
    return True

# Example usage in a CI/CD pipeline
def before_deploy_pipeline(service_a, service_b):
    if not is_compatible(service_a.version, service_b.version):
        raise Exception("Deployment blocked: Incompatible versions")
```

**Example input / output:**

```text
Input:
- service1 version: 1.2.3
- service2 version: 2.0.1

Output:
Deployment allowed: Version pair (1.2.3, 2.0.1) is compatible.
```

**Data:**

```json
{
  "services": [
    { "name": "service1", "version": "1.2.3" },
    { "name": "service2", "version": "2.0.1" }
  ],
  "compatibility_matrix": {
    "1.2.3": ["2.0.1", "2.1.0"],
    "2.0.1": ["1.2.3", "2.1.0"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Service A Version
4. Is Compatible?
5. Proceed to Deployment
6. Block Deployment & NotifyDevs

