---
type: "practice"
difficulty: "Deep"
---

# You need to run a StatefulSet database with three replicas on Kubernetes. Explain how you'd guarantee stable network identity ordered startup and that a re-scheduled pod reattaches to its own PVC and not another replica's data.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Leader Election](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Idempotent_Ingestion/Leader_Election.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure stable network identity, ordered startup, and proper PersistentVolumeClaim (PVC) attachment when rescheduling pods in a StatefulSet database running on Kubernetes, we must carefully design our deployment strategy, configuration, and monitoring. The key is to leverage the inherent properties of StatefulSets and the robustness of Kubernetes networking.

#### Triage

The question probes your understanding of how to manage stateful applications in Kubernetes, specifically focusing on network stability, ordered pod startup, and consistent PVC attachment even during rescheduling events. Key assumptions include:

You have a basic understanding of Kubernetes networking.
You are familiar with StatefulSets and their use cases.
Your solution should handle potential node failures gracefully.

#### First Principles Solution
StatefulSet Configuration: Ensure the StatefulSet manifest is correctly configured to guarantee stable network identity using labels and selectors.
Ordered Startup: Use a custom script or init container to ensure pods start in order based on their stateful index.
PVC Attachment: Configure PVCs with appropriate finalizers to prevent accidental detachment and reattachment.
Step 1: StatefulSet Manifest

Here’s an example StatefulSet manifest snippet:

YAML
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "database-svc"
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: database
        image: your-database-image
        volumeMounts:
        - mountPath: /var/lib/database
          name: database-storage
        command: ["sh", "-c"]
        args: [
            "while [ ! -f /var/lib/database/config-initialized ]; do sleep 5; done;"
        ]
      initContainers:
      - name: wait-for-config
        image: busybox
        command: ["sh", "-c", "until nslookup $(hostname); do echo waiting for DNS; sleep 5; done"]
Step 2: Ordered Startup with Init Containers

The initContainers ensure that the primary containers only start after the database configuration is initialized, maintaining order.

Step 3: PersistentVolumeClaims (PVCs)

Configure PVCs to automatically reattach when a pod is rescheduled:

YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc-0
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  finalizers:
    - kubernetes.io/pvc-protection
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  finalizers:
    - kubernetes.io/pvc-protection
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc-2
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  finalizers:
    - kubernetes.io/pvc-protection

#### Closing Summary
and Common Mistakes/Follow-Ups

To summarize, the key to ensuring stable network identity and consistent PVC attachment is through proper configuration of StatefulSet and PersistentVolumeClaims. You must ensure that pods start in order using init containers and use finalizers on PVCS to prevent accidental detachment. Be wary of common mistakes such as not setting appropriate labels or selectors in StatefulSets, improper handling of PVC lifecycles, and forgetting to configure DNS resolution for network stability.

For follow-ups, you might discuss:

How to handle transient failures and rescheduling.
Additional monitoring and logging practices.
Scalability considerations when increasing the number of replicas.

**Code:**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "database-svc"
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      initContainers:
      - name: wait-for-config
        image: busybox
        command: ["sh", "-c", "until nslookup $(hostname); do echo waiting for DNS; sleep 5; done"]
      containers:
      - name: database
        image: your-database-image
        volumeMounts:
        - mountPath: /var/lib/database
          name: database-storage
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc-0
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  finalizers:
    - kubernetes.io/pvc-protection
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  finalizers:
    - kubernetes.io/pvc-protection
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc-2
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  finalizers:
    - kubernetes.io/pvc-protection
```

