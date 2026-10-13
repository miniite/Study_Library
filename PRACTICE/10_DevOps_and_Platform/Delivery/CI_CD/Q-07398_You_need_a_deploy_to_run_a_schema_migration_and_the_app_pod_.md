---
type: "practice"
difficulty: "Deep"
---

# You need a deploy to run a schema migration and the app pod that runs it as an init job races with a second replica that also tries to migrate; how would you guarantee exactly one migration runs per deploy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Database Migrations in Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Database_Migrations_in_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure exactly one schema migration runs per deployment in your CI/CD pipeline, we need to address two main challenges: coordinating the init container and pod replica sets so that only one migration process is active at a time, and ensuring that any race conditions between them are resolved. This involves leveraging Kubernetes features like init containers and customizing deployment strategies.

#### Triaging the Question

The question revolves around managing schema migrations in a CI/CD pipeline to prevent race conditions where multiple replicas try to perform the same task simultaneously. The core issue is to design a mechanism that ensures only one instance of the migration runs, even when multiple pods are spun up as part of a deployment.

#### First-Principles Solution

The solution hinges on using Kubernetes features effectively and employing a stateful approach to ensure idempotent operations. Specifically, we will use:

Init Containers: To run the schema migration before the application starts.
PodDisruptionBudgets (PDB): To maintain desired pod replicas in the system during disruptions.
State Management: Using a shared state store like a database or etcd to manage the state of migrations.
Step-by-Step Solution
1. Define the Application Deployment

First, we need to define our application deployment with an init container that runs the schema migration script before the main application pod starts.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      initContainers:
      - name: schema-migration
        image: my-schema-migration-image
        command: ["/bin/sh", "-c"]
        args:
          - |
            if ! wget -q "https://s3.amazonaws.com/my-bucket/migrate.sql"; then
              exit 10
            fi
            psql -U postgres -f migrate.sql
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      containers:
      - name: app-pod
        image: my-app-image
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        emptyDir: {}
2. Use PodDisruptionBudget (PDB)

A PDB ensures that a minimum number of replicas are available during disruptions, which is crucial for maintaining the state of the migration process.

YAML
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: my-app
3. Implement State Management

To prevent race conditions, we can use a shared state store like etcd to track the migration status. Each pod will check this state before proceeding with the migration.

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      initContainers:
      - name: schema-migration
        image: my-schema-migration-image
        command: ["/bin/sh", "-c"]
        args:
          - |
            if ! wget -q "https://s3.amazonaws.com/my-bucket/migrate.sql"; then
              exit 10
            fi

            # Check migration state in etcd
            ETCDCTL_API=3 /usr/local/bin/etcdctl get my-app/migration-status || exit 10

            psql -U postgres -f migrate.sql
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      containers:
      - name: app-pod
        image: my-app-image
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        emptyDir: {}

#### Closing Summary
and Common Mistakes/Follow-Ups

By using init containers, PodDisruptionBudgets, and state management via etcd, we can ensure that only one migration runs per deployment. This approach prevents race conditions and ensures idempotent operations.

#### Common Mistakes

Ignoring State Management: Not using a shared state store like etcd to manage the migration status.
Incorrect Init Container Timing: Failing to run init containers before application pods start, leading to potential race conditions.
Insufficient PDB Configuration: Not configuring PDB correctly to maintain desired pod replicas during disruptions.

#### Follow-Ups

How would you handle large schema migrations that might take a long time?
What happens if the migration fails? How do you handle retries or rollbacks?

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      initContainers:
      - name: schema-migration
        image: my-schema-migration-image
        command: ["/bin/sh", "-c"]
        args:
          - |
            if ! wget -q "https://s3.amazonaws.com/my-bucket/migrate.sql"; then
              exit 10
            fi

            # Check migration state in etcd
            ETCDCTL_API=3 /usr/local/bin/etcdctl get my-app/migration-status || exit 10

            psql -U postgres -f migrate.sql
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      containers:
      - name: app-pod
        image: my-app-image
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        emptyDir: {}
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: app-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: my-app
```

