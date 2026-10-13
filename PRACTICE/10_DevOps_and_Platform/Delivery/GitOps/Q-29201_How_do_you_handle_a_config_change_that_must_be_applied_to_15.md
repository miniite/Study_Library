---
type: "practice"
difficulty: "Deep"
---

# How do you handle a config change that must be applied to 150 clusters, some of which are temporarily unreachable, while keeping git as truth?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reconciliation Failures](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this GitOps scenario, we need to manage configuration changes across multiple Kubernetes clusters (150 in total) where some might be temporarily unreachable. The core challenge is ensuring that only reachable clusters receive updates while maintaining the integrity and version control provided by git.

#### Triage
of the Question

The question revolves around understanding how to implement a robust process for applying configuration changes, considering partial availability issues. We need to ensure that:

Changes are applied correctly to all reachable clusters.
Git remains the single source of truth for configurations.
Temporarily unreachable clusters are not updated but their state is preserved.

#### Assumptions
Each cluster has a corresponding repository in git containing its configuration files.
Clusters can be in one of three states: online, offline (temporarily unreachable), or failed.
Configuration changes need to be applied consistently across all online clusters.
The process should handle failures gracefully and not update any partially failing clusters.

#### First-Principles Solution

To achieve this, we'll leverage GitOps principles by:

Defining a Deployment Strategy: Use a declarative approach where the desired state of each cluster is defined in its repository.
Monitoring Cluster Status: Implement a health check mechanism to determine if a cluster is online or offline.
Selective Application: Apply changes only to online clusters, ensuring that offline clusters remain unchanged but their configurations are updated once they come back online.
Rollback Mechanism: Include mechanisms for rolling back changes in case of failures.
Step-by-Step Solution

Repository Setup:

Each cluster has a corresponding repository containing its configuration files (e.g., Helm charts, Kubernetes manifests).
BASH
# Example structure for a cluster repository
repo_name/
├── config/
│   └── helm-charts/
│       └── values.yaml
└── scripts/
    └── apply-config.sh

Health Check Mechanism:

Create a script that checks the availability of each cluster.
BASH
# Example health check script
#!/bin/bash

for CLUSTER in $(kubectl get nodes -o name | cut -d '/' -f 3); do
  if ! kubectl --context=$CLUSTER get po > /dev/null; then
    echo "Cluster $CLUSTER is offline"
  else
    echo "Cluster $CLUSTER is online"
  fi
done

Selective Application:

Modify the apply-config.sh script to only apply changes to online clusters.
BASH
#!/bin/bash

# Get list of online clusters from health check
ONLINE_CLUSTERS=$(health_check_script.sh)

for CLUSTER in $ONLINE_CLUSTERS; do
  kubectl --context=$CLUSTER apply -f config/helm-charts/values.yaml
done

Rollback Mechanism:

Implement a rollback strategy using Git tags and branches.
BASH
# Example rollback script
#!/bin/bash

LAST_TAG=$(git describe --abbrev=0 --tags)
NEW_VERSION="v2"

git checkout $LAST_TAG
git commit -am "Rollback to previous version"
git tag -d $NEW_VERSION
git push origin --delete $NEW_VERSION

Continuous Integration/Continuous Deployment (CI/CD):

Integrate the above scripts into a CI/CD pipeline that automatically applies changes and handles rollbacks.
YAML
# Example GitHub Actions workflow
name: Apply Config Changes

on:
  push:
    branches:
      - main

jobs:
  apply-changes:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Health Check
        run: ./health_check_script.sh

      - name: Apply Config to Online Clusters
        run: ./apply-config.sh

      - name: Rollback on Failure
        if: ${{ failure() }}
        run: ./rollback_script.sh

#### Closing Summary
and Common Mistakes/Follow-Ups

In this solution, we ensure that configuration changes are applied only to online clusters while maintaining the integrity of unreachable clusters. The key is to use Git as the single source of truth and implement robust health checks and rollback mechanisms.

#### Common Mistakes

Failing to distinguish between online and offline clusters.
Not having a reliable mechanism for applying changes selectively.
Ignoring the need for a rollback strategy in case of failures.

#### Follow-Ups

How would you handle rollbacks if multiple versions are involved?
What additional metrics would you track to ensure high availability?
Can you integrate this with existing monitoring tools like Prometheus or Grafana?

**Comparison:**

```
Cluster Status	Action Taken
Online	Apply configuration
Offline	Skip application
Failed	Log failure and skip
```

**Code:**

```bash
# Example health check script
#!/bin/bash

for CLUSTER in $(kubectl get nodes -o name | cut -d '/' -f 3); do
  if ! kubectl --context=$CLUSTER get po > /dev/null; then
    echo "Cluster $CLUSTER is offline"
  else
    echo "Cluster $CLUSTER is online"
  fi
done

# Example apply-config script
#!/bin/bash

ONLINE_CLUSTERS=$(health_check_script.sh)

for CLUSTER in $ONLINE_CLUSTERS; do
  kubectl --context=$CLUSTER apply -f config/helm-charts/values.yaml
done

# Example rollback script
#!/bin/bash

LAST_TAG=$(git describe --abbrev=0 --tags)
NEW_VERSION="v2"

git checkout $LAST_TAG
git commit -am "Rollback to previous version"
git tag -d $NEW_VERSION
git push origin --delete $NEW_VERSION
```

**Example input / output:**

```text
# Example input/output for health check script
Cluster cluster-1 is online
Cluster cluster-2 is offline
Cluster cluster-3 is online

# Example output from apply-config script
Cluster cluster-1 is online
Applying config to cluster-1...
Config applied successfully.
```

**Data:**

```yaml
# Example GitHub Actions workflow
name: Apply Config Changes

on:
  push:
    branches:
      - main

jobs:
  apply-changes:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Health Check
        run: ./health_check_script.sh

      - name: Apply Config to Online Clusters
        run: ./apply-config.sh

      - name: Rollback on Failure
        if: ${{ failure() }}
        run: ./rollback_script.sh
```

**Flow:**

1. Yes
2. No
3. Health Check
4. Is Cluster Online?
5. Apply Configuration
6. Log as Offline
7. Rollback Mechanism

