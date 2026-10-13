---
type: "practice"
difficulty: "Intermediate"
---

# Your team's pipeline builds images on every commit and your registry storage cost is now the biggest line item; how would you set retention, tag hygiene, and dedup so you keep what you need but stop the bleed?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Container Images and Registries](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Container_Images_and_Registries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of excessive storage costs due to frequent image builds in a CI/CD pipeline, we need to optimize our approach by implementing effective retention policies, ensuring proper tag hygiene, and leveraging deduplication strategies. This will ensure that we retain only the necessary images while minimizing unnecessary storage.

Triaging the Problem

The core question revolves around managing the lifecycle of Docker images stored in a container registry. The key aspects to consider are:

Retention Policies: Define how long each image should be retained.
Tag Hygiene: Ensure that tags are managed efficiently and logically.
Deduplication: Reduce storage by storing only unique images.

#### Assumptions
We use a container registry like Docker Hub or ECR.
The CI/CD pipeline triggers builds on every commit.
The goal is to reduce storage costs without compromising the availability of necessary images.

#### First Principles Solution
1. Retention Policies

To implement retention policies, we need to define how long each image should be retained based on its importance and usage frequency. Here’s a step-by-step approach:

Short-Term Retention (e.g., 1 day): Store recent builds for testing purposes.
Medium-Term Retention (e.g., 30 days): Keep images for production use, ensuring there are no breaks in service due to image issues.
Long-Term Retention (e.g., 90 days): Archive older versions of images for compliance or historical reasons.

We can achieve this using a combination of CI/CD pipeline triggers and registry hooks. For example, we could set up a cron job that automatically tags new builds with a specific short-term tag, medium-term tag, and long-term tag. This way, the images will be stored in different layers of the registry.

BASH
# Example shell script to tag images
if [ $(git rev-list --count HEAD) -le 10 ]; then
    docker tag <image> <registry>/<repo>:latest:short_term
    docker tag <image> <registry>/<repo>:latest:medium_term
    docker tag <image> <registry>/<repo>:latest:long_term
fi
2. Tag Hygiene

Proper tag management ensures that tags are meaningful and not redundant, which helps in reducing storage costs by avoiding unnecessary duplication.

Use Semantic Versioning: Ensure tags follow a consistent versioning scheme (e.g., v1.0, v1.0.1, v2.0).
Canonical Tags: Use canonical tags like latest, stable, and release to point to the most recent versions.
Avoid Redundant Tags: Ensure that each image has a unique set of tags, and avoid tagging with multiple identical tags.

For example, in Docker:

BASH
docker tag <image> <registry>/<repo>:v1.0
docker tag <image> <registry>/<repo>:latest
3. Deduplication

Deduplication is a critical step to reduce storage costs by storing only unique images and leveraging multi-architecture support.

Multi-Architecture Support: Use buildx in Docker to create multi-platform builds that can be stored once but used across multiple architectures.
Tag Aliases: Create aliases for tags to point to the most recent version, reducing redundant storage of identical images.
BASH
docker buildx build --platform linux/amd64,linux/arm64 -t <repo>:latest .

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, by implementing a combination of retention policies, tag hygiene, and deduplication strategies, we can significantly reduce storage costs while maintaining the necessary images for production use.

#### Common mistakes
include:

Failing to establish a clear tagging strategy.
Not using multi-architecture support, leading to redundant storage.
Ignoring long-term retention policies, potentially losing important historical versions.

For follow-ups, you might want to explore more advanced techniques such as image layering optimization and leveraging container orchestration platforms like Kubernetes for further cost savings.

**Comparison:**

```
Tag	Description
latest	Points to the most recent version for quick deployment.
v1.0	Specific version tag for v1.0 release.
v2.0	Specific version tag for v2.0 release.
Alias	Target Tag	Description
staging	latest	For development and testing environments.
production	v2.0	For production use.
```

**Code:**

```bash
# Example of a CI/CD script snippet
if [ $(git rev-list --count HEAD) -le 10 ]; then
    docker tag <image> <registry>/<repo>:latest:short_term
    docker tag <image> <registry>/<repo>:latest:medium_term
    docker tag <image> <registry>/<repo>:latest:long_term
fi

docker buildx build --platform linux/amd64,linux/arm64 -t <repo>:latest .
```

**Example input / output:**

```text
# Example input and output for tagging process
Input:
- Commit with new image

Output:
- Docker images tagged as <registry>/<repo>:latest:short_term, <registry>/<repo>:latest:medium_term, <registry>/<repo>:latest:long_term

Docker commands executed:
```

**Data:**

```json
{
  "retention_policies": {
    "short_term": {"days": 1},
    "medium_term": {"days": 30},
    "long_term": {"days": 90}
  },
  "tagging_strategy": {
    "latest": true,
    "versioned": ["v1.0", "v2.0"],
    "aliases": [
      {"alias": "staging", "target": "latest"},
      {"alias": "production", "target": "v2.0"}
    ]
  }
}
```

**Flow:**

1. CI/CD Pipeline Trigger
2. Tagging Script
3. Short-Term Tag:<image>:latest:short_term
4. Medium-Term Tag:<image>:latest:medium_term
5. Long-Term Tag:<image>:latest:long_term
6. Mult-Arch Build: --platformlinux/amd64,linux/arm64 -t<repo>:latest

