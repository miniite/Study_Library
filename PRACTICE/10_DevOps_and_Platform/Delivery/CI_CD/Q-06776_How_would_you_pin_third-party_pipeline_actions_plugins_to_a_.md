---
type: "practice"
difficulty: "Deep"
---

# How would you pin third-party pipeline actions/plugins to a specific SHA and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Secrets in Pipelines](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Secrets_in_Pipelines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In CI/CD pipelines, "pinning" third-party pipeline actions or plugins to a specific SHA (Secure Hash Algorithm) ensures that your build process uses a known, stable version of those dependencies, even if newer versions are available or if changes introduce issues. This is crucial for maintaining consistency and reliability across deployments.

To address the question from first principles:

Understanding the Question: The interviewer is probing how you would enforce specific versions of third-party tools in your CI/CD pipeline.

#### Assumptions
You are using a common CI/CD tool like Jenkins, GitLab CI, or GitHub Actions.
Third-party actions/plugins are invoked via script or configuration files.
Step-by-Step Solution
1. Identify the Specific SHA

First, determine which specific version of the third-party action/plugin you want to pin down. You might have a good reason for choosing this particular commit, such as stability concerns, known bugs, or performance improvements in previous versions.

2. Modify Your CI/CD Configuration File

In your pipeline configuration file (e.g., Jenkinsfile, .gitlab-ci.yml, or .github/workflows/<file>.yml), specify the exact version of the third-party action/plugin to use.

For example, if you are using a GitHub Action and want to pin it to SHA abcdef1234567890, your configuration might look like this:

YAML
steps:
  - name: Run custom script
    uses: user/repo@abcdef1234567890
3. Ensure Version Compatibility

Check that the pinned version is compatible with your project and does not introduce any breaking changes or issues.

4. Test Your Pipeline

Run your CI pipeline manually to ensure that it uses the correct version of the third-party tool and works as expected. This step helps catch any issues early before deploying to production.

5. Document Your Changes

Update your project documentation to reflect this change, noting the reasons for pinning a specific version and the exact SHA used.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
Pinning third-party pipeline actions or plugins to a specific SHA ensures that your builds are reproducible and stable. It is crucial for maintaining consistency in development environments and avoiding potential issues from newer, untested versions of dependencies.

#### Common Mistakes

Forgetting to Test: Always test the pinned version thoroughly before deploying.
Not Documenting: Failing to document which SHA was used can lead to confusion later.
Inadequate Version Management: Not keeping track of why a specific version is pinned, making it difficult to revert or update in the future.

#### Follow-Ups
The interviewer might ask about alternative strategies for managing third-party dependencies or how you handle security updates for these plugins.

**Code:**

```yaml
steps:
  - name: Run custom script
    uses: user/repo@abcdef1234567890
```

