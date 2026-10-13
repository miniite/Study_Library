---
type: "practice"
difficulty: "Deep"
---

# A GitOps rollback by reverting the commit didn't restore service because the old image tag had been garbage-collected from the registry — how do you make rollbacks truly reliable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Resilience to Git Outages](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In a GitOps environment, rolling back changes by reverting commits often results in services not returning to their previous state due to issues like image garbage collection from container registries. To ensure true reliability in rollbacks, we need to implement a comprehensive strategy that includes versioning and retention policies for images, and possibly a staged rollback process.

#### Triaging the Question

The question is probing how to reliably manage deployments and rollbacks in a GitOps environment. Specifically, it highlights the issue where reverting a commit did not restore the service because an old image tag was garbage-collected from the registry. This situation underscores the need for robust versioning practices and possibly additional safeguards.

#### Assumptions
The GitOps workflow involves deploying code changes to production through Git repositories.
Container registries are used to store Docker images.
Garbage collection policies in container registries automatically remove old tags or images based on time or space constraints.

#### First-Principles Solution

To ensure reliable rollbacks, we need a combination of robust deployment strategies and effective governance over our container registry:

1. Versioning and Tagging Images

Implement a consistent versioning strategy for your Docker images. This can be done using semantic versioning (e.g., v2.3.0), where the version number is incremented according to specific rules.

BASH
# Example tagging script in Bash
IMAGE_TAG=$(git describe --tags --always)
docker tag myapp:$IMAGE_TAG myregistry/myapp:$IMAGE_TAG
2. Retention Policies for Docker Registries

Configure your container registry with retention policies that keep old versions of images available, even after they are tagged as "latest" or removed from the GitOps repository.

Docker Registry Policy: Set up a policy to retain tags based on age and/or size.
YAML
version: 0.1
rules:
  - if: Type==\"image\"
    when:
      exists: /path/to/timeline.json
    then:
      keep: "24h"  # Keep images for 24 hours before garbage collection
  - if: Type==\"image\"
    when:
      exists: /path/to/budget.txt
    then:
      keep: "10GB"  # Keep at least 10 GB of image storage
3. Staged Rollbacks

Implement a staged rollback process where the service is first rolled back to an earlier version, allowing time for manual verification and any necessary adjustments before fully switching to the previous state.

BASH
# Example staged rollback script in Bash
echo "Rolling back to $OLD_IMAGE_TAG"
docker pull myregistry/myapp:$OLD_IMAGE_TAG
docker stop $(docker ps -q --filter label=app=myapp)
docker rm $(docker ps -aq --filter label=app=myapp)
docker run -d --name myapp --label app=myapp -p 80:80 $OLD_IMAGE_TAG
4. Monitoring and Alerts

Set up monitoring and alerting systems to notify the team when an image is about to be garbage-collected or has been removed, allowing for timely action.

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, making rollbacks truly reliable in a GitOps environment involves implementing versioned tagging, robust retention policies, staged rollback procedures, and effective monitoring. Common mistakes include neglecting image versioning, not configuring appropriate registry retention policies, and failing to test the rollback process thoroughly.

For follow-ups:

Discuss how automated tools like Helm can be used for managing versions and rollbacks.
Explore hybrid cloud strategies where multiple registries are involved and need coordination.

#### Cons
ider CI/CD pipeline optimizations to reduce deployment times and improve reliability.

**Comparison:**

```
Version	Tagging Strategy	Retention Policy	Monitoring
1	Semantic versioning (e.g., v2.3.0)	Keep images for 24 hours, retain at least 10GB of storage	Notify on image removal or garbage collection
```

**Code:**

```bash
# Example tagging script
IMAGE_TAG=$(git describe --tags --always)
docker tag myapp:$IMAGE_TAG myregistry/myapp:$IMAGE_TAG

# Staged rollback script
echo "Rolling back to $OLD_IMAGE_TAG"
docker pull myregistry/myapp:$OLD_IMAGE_TAG
docker stop $(docker ps -q --filter label=app=myapp)
docker rm $(docker ps -aq --filter label=app=myapp)
docker run -d --name myapp --label app=myapp -p 80:80 $OLD_IMAGE_TAG
```

**Example input / output:**

```text
Input: Reverting commit to previous state in GitOps repository

Output:
- Docker registry retains old images based on policy
- Staged rollback process ensures manual verification before full switch
- Service returns to previous working version
```

**Data:**

```yaml
# Example Docker registry policy configuration
version: 0.1
rules:
  - if: Type==\"image\"
    when:
      exists: /path/to/timeline.json
    then:
      keep: "24h"  # Keep images for 24 hours before garbage collection
  - if: Type==\"image\"
    when:
      exists: /path/to/budget.txt
    then:
      keep: "10GB"  # Keep at least 10 GB of image storage
```

**Flow:**

1. Revert Git commit
2. Tag old image
3. Push to registry
4. Configure retention policy
5. Monitor images for deletion
6. Stage rollback, verify manually
7. Switch back to previousversion if verified

