---
type: "practice"
difficulty: "Intermediate"
---

# Artifacts uploads are slow and hit size limits. How do you reduce artifact bloat while keeping what you need?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Docker Layer Caching](<../../../../TOOLS/GitLab/GitLab_CI/Docker_Layer_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of slow artifact uploads and size limits in GitLab CI, we need to focus on reducing artifact bloat while ensuring that critical artifacts are retained. This involves a combination of artifact filtering, optimization, and leveraging GitLab's features effectively.

#### Triaging the Question

The question is asking how to manage GitLab CI artifacts more efficiently to reduce their size and improve upload times without compromising the essential data needed for reproducibility and traceability. The interviewer may be probing your understanding of GitLab CI best practices, artifact management strategies, and performance optimization techniques.

#### Assumptions
We are dealing with a multi-stage CI/CD pipeline.
The artifacts in question include binaries, logs, and other non-essential files.
There is a need to balance the trade-off between keeping necessary data and reducing overhead.

#### Solution
from First Principles
Step 1: Artifact Filtering

Start by filtering out unnecessary files during the build process. GitLab CI supports wildcard patterns for excluding specific files or directories in your .gitlab-ci.yml configuration.

YAML
# Example of artifact filtering
stages:
  - build
  - test
  - deploy

build_job:
  stage: build
  script:
    - make
  artifacts:
    paths:
      - target/ # Keep necessary binaries
      - logs/   # Keep essential logs for debugging
    except:
      - '**/*.log' # Exclude all log files from artifact upload (keep just critical ones)
Step 2: Compressing Artifacts

Compress the retained artifacts before uploading. GitLab CI supports various compression formats like tar, zip, and gzip to reduce the size of the uploaded data.

YAML
# Example of compressing artifacts
after_script:
  - tar czf artifacts.tar.gz target/ logs/
artifacts:
  paths:
    - artifacts.tar.gz # Compressed artifact for upload
Step 3: Using GitLab Pages or External Storage

For very large artifacts, consider using GitLab Pages to serve static content directly from the repository. Alternatively, store larger data in external cloud storage like AWS S3 and reference it within your pipeline.

YAML
# Example of using external storage
artifacts:
  paths:
    - target/
  expire_in: 1 week # Retain artifacts for a week

deploy_job:
  stage: deploy
  script:
    - aws s3 cp --recursive s3://my-bucket/path/to/artifacts/ target/
Step 4: Monitoring and Optimization

Continuously monitor artifact sizes and upload times. Use GitLab’s built-in monitoring tools or integrate with external metrics to identify bottlenecks.

#### Closing Summary

By filtering, compressing, and selectively storing artifacts, we can significantly reduce the size of uploaded data in GitLab CI. This not only speeds up uploads but also optimizes storage costs and improves overall pipeline performance. It's crucial to regularly review these configurations as your project evolves and new requirements emerge.

#### Common Mistakes
and Follow-Ups
Not filtering out unnecessary files: Keeping all files can lead to bloated artifacts.
Over-compressing data: Compression might introduce additional overhead, so it’s important to balance between size reduction and performance.
Ignoring external storage options: For very large datasets, leveraging cloud storage solutions can be more efficient than上传内容已结束，请完成剩余部分。 assistant 内部存储。

#### Common Mistakes
and Follow-Ups
Not filtering out unnecessary files: Keeping all files can lead to bloated artifacts.
Over-compressing data: Compression might introduce additional overhead, so it’s important to balance between size reduction and performance.
Ignoring external storage options: For very large datasets, leveraging cloud storage solutions can be more efficient than internal GitLab storage in terms of cost and scalability.

#### Follow-Ups

How would you handle different types of artifacts (e.g., binaries vs. logs)?

Differentiate between critical and non-critical files based on their nature and use case. For example, keep all essential binary outputs but compress or exclude log files that can be archived separately.

What if the artifact size is still too large?

#### Cons
ider using a content-addressable storage solution like Git Annex to store large files outside of the repository while keeping metadata in GitLab.

How do you ensure data integrity during transfers and storage?

Use checksums or hashing functions to verify file integrity before and after uploads. For instance, add sha256sum checks in your pipeline to validate that uploaded artifacts match expected values.

What about artifact retention policies?

Define clear retention policies based on the lifecycle of your projects. This could involve setting up automatic cleanup or archiving older versions of artifacts after a certain period.
Additional Strategies

Customizing Artifacts Based on Stage: Different stages may require different sets of artifacts, so tailor artifact configurations per stage to optimize each step.

YAML
# Example of customized artifacts for different stages
build_job:
  artifacts:
    paths:
      - target/
  expire_in: 1 week

test_job:
  artifacts:
    paths:
      - test-reports/
  expire_in: 24 hours

deploy_job:
  artifacts:
    paths:
      - production-config/
  expire_in: 7 days

Leveraging GitLab CI/CD Variables: Use environment variables to dynamically control artifact paths and configurations based on different environments or branches.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, optimizing GitLab CI artifacts involves a multi-faceted approach that includes filtering, compression, external storage, and continuous monitoring. By carefully managing these aspects, you can significantly reduce bloat while ensuring the necessary data is retained for reproducibility and traceability.

#### Common mistakes
include not filtering out unnecessary files, over-compressing which might introduce overhead, ignoring external storage options, and failing to implement effective artifact retention policies.

Always keep an eye on performance metrics and adjust your strategies as needed based on project requirements and evolving best practices in CI/CD.

