---
type: "practice"
difficulty: "Deep"
---

# How would you design a promotion pipeline where the exact bytes that passed staging are what deploy to prod, and what breaks that guarantee if someone re-runs the build job?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Build Reproducibility and Artifact Promotion](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/Build_Reproducibility_and_Artifact_Promotion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a robust promotion pipeline ensuring that only the exact bytes from the staging environment are deployed to production, we need to focus on immutable builds and consistent deployment strategies. The core idea is to ensure that any changes in the build process do not alter the artifact being deployed. Here’s how we can achieve this:

Immutable Builds: Ensure that once a build is created, it remains unchanged.
Versioned Artifacts: Use versioning for artifacts to track specific builds.
Staging Mirror: Deploy an exact copy of the staging environment to ensure consistency before production deployment.

In this design, we will use first-principles reasoning to craft a solution that guarantees the integrity of the deployed code and ensures that re-running build jobs does not break this guarantee.

**Comparison:**

```
Version	Timestamp	SHA256
v123	2023-10-05	abcde1234fghij56789klmno9012pqr345stuv6789wxyz
```

**Code:**

```python
# Example of a simple versioning strategy using Git tags for artifacts
def generate_tag(version_number):
    return f"v{version_number}"

# Sample usage to tag a specific commit
tag = generate_tag(123)
print(f"Generated tag: {tag}")
```

**Example input / output:**

```text
Input:
- Staging environment is configured and ready.
- A new version of the application is built with Git tag v123.

Output:
- An exact copy of the artifacts from v123 are deployed to production, ensuring no changes were made during the staging process.
```

**Data:**

```yaml
artifacts:
  - version: v123
    timestamp: "2023-10-05T14:00Z"
    sha256: "abcde1234fghij56789klmno9012pqr345stuv6789wxyz"
```

**Flow:**

1. Yes
2. No
3. Staging Build
4. Generate Git Tag v123
5. Deploy Artifacts to Staging
6. Verify Staging Environment
7. Run Production Deploy?
8. Fetch Exact Artifacts fromStaging
9. Deploy Artifacts to Production
10. Re-run Build Job
11. Mutate Artifacts in Staging
12. Staging Deviation Detected

