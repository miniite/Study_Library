---
type: "practice"
difficulty: "Deep"
---

# You want to prove to auditors which exact commit of the alerting rules and remediation runbooks is running across five monitoring clusters at any moment. Compare baking the SHA into the image, exposing it via a probe endpoint, and reading it from the GitOps sync status, and which is authoritative.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reproducible Builds from Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reproducible_Builds_from_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to ensure that auditors can verify exactly which commit of alerting rules and remediation runbooks are currently deployed across multiple monitoring clusters. We'll compare three methods: baking the SHA into the image, exposing it via a probe endpoint, and reading it from the GitOps sync status. Each approach has its own trade-offs in terms of reliability, maintainability, and ease of verification.

#### Triage

The question is essentially asking us to evaluate the best way to ensure traceability and accountability for the code running across multiple monitoring clusters. The key aspects include:

Traceability: Ensuring that every instance of a deployed image can be linked back to its corresponding Git commit.
Auditing: Providing verifiable evidence during audits.
Maintainability: Managing changes in the CI/CD pipeline and ensuring that deployments are consistent.

#### Assumptions
We have access to monitoring clusters, images, and Git repositories.
We can integrate custom probes or metadata collection mechanisms within our deployment process.

#### First-Principles Solution
1. Baking the SHA into the Image

#### Pros

Direct Linkage: The SHA is baked directly into the image at build time, ensuring that there's no ambiguity about which commit corresponds to a deployed version.
Simplicity: Once an image is tagged with its commit SHA, it’s straightforward to verify.

#### Cons

Complexity in Build Process: Requires modifying the build process to ensure every image is tagged correctly.
Manual Verification: While the SHA can be read from the image metadata, this still requires manual or automated steps to check against Git history.
2. Exposing the SHA via a Probe Endpoint

#### Pros

Dynamic Verification: The probe endpoint provides real-time verification of the running code version without requiring direct access to the image.
Simplified Image Build Process: No need to bake metadata directly into the image; just expose it through an endpoint.

#### Cons

Reliability: Network issues or misconfigurations in the probe might cause false negatives during audits.
Performance Overhead: Continuous querying of the probe can introduce additional latency and load on the monitoring clusters.
3. Reading SHA from GitOps Sync Status

#### Pros

Automated Verification: The GitOps sync status automatically updates to reflect the latest deployment, providing a seamless way to verify code versions.
Integrated Workflow: Fits well with existing CI/CD pipelines and tools used for versioning and deployment.

#### Cons

Dependency on Infrastructure: Relies heavily on the reliability of the underlying GitOps infrastructure (e.g., Flux, Argo CD).
Potential Lag: There might be a delay between when code is committed to Git and when it appears in the deployed clusters due to sync latencies or network issues.

#### Closing Summary

Given these trade-offs, reading the SHA from the GitOps sync status appears to offer the best balance of reliability and ease of maintenance. However, it’s crucial to ensure that this approach integrates well with existing monitoring and auditing practices.

#### Common Mistakes/Follow-Ups
Manual Verification: Ensure that manual steps for verifying image SHAs are clearly documented.
Probing Issues: Implement robust error handling in the probe endpoint to minimize false negatives during audits.
Latency Management: Optimize GitOps sync times to reduce potential delays between code commits and deployments.

**Comparison:**

```
Method	Pros	Cons
Baking into Image	Direct and clear linkage between image and commit.	Requires modifying build process; manual verification needed.
Probe Endpoint	Dynamic and real-time verification without access to images.	Potential for network issues or misconfigurations leading to false negatives.
GitOps Sync Status	Automated and integrated with CI/CD, less prone to human error in verification.	Heavy reliance on underlying infrastructure; potential sync lag.
```

**Code:**

```python
# Example Python script to read SHA from a Docker image metadata
def get_image_sha(image_metadata):
    return image_metadata['sha']
```

**Example input / output:**

```text
Input:
- Deployed image with metadata {"sha": "abc123", "tag": "v1.0.0"}

Output:
- Read SHA from image: abc123
- Linked to Git commit: https://github.com/organization/repo/commit/abc123
```

**Data:**

```json
{
  "image_metadata": {
    "sha": "abc123",
    "tag": "v1.0.0"
  }
}
```

**Flow:**

1. Code Commit
2. Built Image
3. Tagged Image
4. Deployed Cluster
5. Probe Endpoint Verification
6. GitOps Sync Status

