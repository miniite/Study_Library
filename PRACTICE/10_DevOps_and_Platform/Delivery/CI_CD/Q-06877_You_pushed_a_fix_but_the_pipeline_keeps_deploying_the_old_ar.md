---
type: "practice"
difficulty: "Deep"
---

# You pushed a fix but the pipeline keeps deploying the old artifact; walk me through every layer (cache, tag, image digest, CDN, orchestrator) you'd check to find where the stale version comes from.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ CI Caching](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/CI_Caching.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter an issue with a CI/CD pipeline deploying a stale artifact despite pushing a fix, it's important to systematically investigate each layer of your deployment stack to pinpoint the problem. This process involves checking the cache, tags, image digests, CDN, and orchestrator configurations.

This scenario often arises due to misconfigurations or misunderstandings in how these layers interact. Let’s break down each potential issue step by step:

Cache Layer: Start by examining your caching mechanism. CI/CD tools like Jenkins or GitHub Actions use caches to speed up build processes. Ensure that the cache was invalidated properly after pushing the new artifact.

Tagging and Image Digests: Verify how tags and image digests are managed in your Docker registry. Tags can be a major culprit here, as they might not get updated correctly. Additionally, check if the image digest is being used to uniquely identify the deployed version. Incorrect tag management or outdated digests can lead to deploying an old artifact.

Orchestrator (Kubernetes): If your application is running in Kubernetes, investigate its deployment strategy and any related configurations. Kubernetes uses rolling updates by default but can also use strategies like Recreate or RollingUpdate. Ensure that the correct rollout policy is set to avoid deploying old versions.

CDN Layer: If you are using a CDN (Content Delivery Network) for serving your application, check its configuration and caching settings. CDNs often cache content aggressively, so ensure that they have been invalidated properly or that the new version has propagated through the CDN.

Manual and Automated Checks: Perform manual checks by inspecting the logs of all relevant components—CI/CD jobs, Docker registry, Kubernetes cluster, and any intermediary tools like Helm charts or custom scripts. Look for signs of stale artifacts being deployed inadvertently.

By systematically working through these layers, you can isolate where the issue lies and take corrective action to ensure that only the correct version is deployed.

#### Common mistakes
include:

Not invalidating caches properly after pushing a new artifact.
Misconfiguring tags in Docker or Helm charts.
Failing to update Kubernetes deployment strategies or rolling updates correctly.
Not revalidating content on CDNs.

**Comparison:**

```
Layer	Potential Issue	Check
Cache	Cache not cleared	docker system prune
Tags	Incorrect tag	git tag -d and docker rmi
Digests	Outdated digest	docker image inspect
Orchestrator	Rollout strategy misconfig	kubectl rollout status
CDN	Content not revalidated	purge-cache command on CDN dashboard
```

**Code:**

```python
# Example of checking cache invalidation in Jenkins Pipeline
def invalidateCache() {
    sh 'docker system prune --all --force'
}
```

**Flow:**

1. Cache
2. Tag Management
3. Docker Registry Digests
4. Kubernetes DeploymentStrategy
5. CDN Configuration
6. Manual and AutomatedChecks

