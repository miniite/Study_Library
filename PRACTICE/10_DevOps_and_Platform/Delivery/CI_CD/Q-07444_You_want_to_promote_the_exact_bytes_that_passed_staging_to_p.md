---
type: "practice"
difficulty: "Deep"
---

# You want to promote the exact bytes that passed staging to prod, but a re-run of the build job produces a slightly different artifact; what breaks the immutability guarantee and how would you enforce build-once-promote-many?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Build Reproducibility and Artifact Promotion](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/Build_Reproducibility_and_Artifact_Promotion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure immutable promotion from staging to production in CI/CD, we need to address the issue where a re-run of the build job produces a slightly different artifact due to minor variations such as timestamps or environment-specific configurations. The key challenge is breaking the immutability guarantee, and enforcing that once an artifact passes staging, it remains unchanged for promotion.

In this scenario, the main issues are:

Environment Differences: Staging vs Production environments might have slight differences (e.g., timestamps).
Build Variations: Re-running a build job can produce artifacts with minor changes due to different build contexts.
Artifact Versioning and Signing: Ensuring that the exact artifact used in staging is also used in production.

To enforce an immutable promotion process, we need to implement several strategies:

Hash-Based Artifact Verification:

Calculate a hash (e.g., SHA-256) of the artifact when it passes staging.
Store this hash as a metadata tag within the deployment record.
Verify that the production deployment uses an artifact with the same hash.

Environment Agnostic Build Jobs:

Standardize build jobs to remove environment-specific configurations and timestamps from the build process.
Use immutable containers or configuration management tools (e.g., Kubernetes ConfigMaps) to handle environment-specific settings post-deployment.

Continuous Integration/Continuous Deployment (CI/CD) Pipeline Configuration:

Ensure that the CI/CD pipeline captures all relevant information about the build context and stores it securely.
Use Git commit timestamps or other unique identifiers in the artifact name to prevent confusion.

Immutable Artifact Staging and Promotion:

Store the final artifact in an immutable repository (e.g., AWS S3, GCS) with version control.
Deploy only from this repository to ensure consistency across environments.

Automated Testing and Validation:

Implement automated tests that run on every build and verify the integrity of the artifact before promotion.
Use pre-deployment checks to validate that the artifact meets all necessary criteria for production use.

By implementing these strategies, we can effectively enforce an immutable promotion process where the exact bytes that passed staging are promoted to production without any changes.

