---
type: "practice"
difficulty: "Deep"
---

# How do you deploy Flink on Kubernetes with the operator, and what changes for an always-on AIOps pipeline versus a batch backfill?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Zero-loss Job Upgrades](<../../../../TOOLS/Apache_Flink/Flink_Core/Zero-loss_Job_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deploying Apache Flink on Kubernetes using an operator offers robust management of stateful stream processing jobs. For an always-on AIOps pipeline, you need to ensure high availability and fault tolerance, while for a batch backfill, the focus shifts towards resource efficiency and job completion speed.

Kubernetes provides a powerful platform for deploying Flink, leveraging its inherent containerization capabilities. The key steps involve setting up the Flink cluster with an operator that manages the lifecycle of your jobs, scaling them according to demand, and ensuring they can recover from failures seamlessly.

#### Triaging the Question
Deployment Mechanism: How do you ensure reliable deployment and management of Flink jobs on Kubernetes?
Operator Role: What is the purpose of using a Kubernetes operator for managing Flink clusters?
Always-On vs Batch Pipeline:
Always-On: Continuous processing with high availability.
Batch Backfill: Offline or historical data processing, often one-time execution.

#### First-Principles Solution
1. Setting Up the Flink Cluster

First, create a Kubernetes deployment for your Flink cluster using a statefulset. This ensures that each task manager has its own persistent storage and can recover from failures:

YAML
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: flink-cluster
spec:
  replicas: 3 # Ensure high availability
  selector:
    matchLabels:
      app: flink
  template:
    metadata:
      labels:
        app: flink
    spec:
      containers:
      - name: flink-taskmanager
        image: apache/flink:1.14.0
        env:
          - name: TASK_MANAGER.numberOfTaskSlots
            value: "3" # Define the number of task slots per task manager
2. Using a Kubernetes Operator for Flink

A Kubernetes operator simplifies the management of complex applications like Apache Flink by abstracting away many of the manual deployment and scaling operations. For instance, the flink-operator is a popular choice that integrates with the Flink cluster to handle job submission, monitoring, and recovery.

Here's an example of how you might configure the operator:

YAML
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: flink-group
spec: {}

---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flinkjobs.flink.apache.org
spec:
  group: flink.apache.org
  versions:
    - name: v1beta1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              properties:
                jobmanagerImage:
                  type: string

---
apiVersion: flink.apache.org/v1beta1
kind: FlinkJob
metadata:
  name: example-job
spec:
  jobManagerImage: apache/flink:1.14.0
3. Always-On AIOps Pipeline

For an always-on AIOps pipeline, focus on high availability and fault tolerance:

High Availability: Use a statefulset with multiple replicas.
Fault Tolerance: Ensure job recovery using checkpointing and savepoints.
Scalability: Set up horizontal pod autoscalers (HPAs) to scale the cluster based on load.
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: flink-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: StatefulSet
    name: flink-cluster
  minReplicas: 3
  maxReplicas: 5
  targetCPUUtilizationPercentage: 50 # Scale based on CPU usage
4. Batch Backfill Pipeline

For batch backfills, the focus shifts to optimizing resource utilization and job completion speed:

Resource Efficiency: Use fewer resources but with higher priority.
Job Completion: Ensure jobs complete quickly by leveraging dedicated resources.
YAML
apiVersion: batch/v1
kind: Job
metadata:
  name: flink-backfill-job
spec:
  template:
    spec:
      containers:
      - name: flink-taskmanager
        image: apache/flink:1.14.0
        command: ["/bin/sh"]
        args: ["-c", "flink run --job-manager localhost:8081 /path/to/your/job.jar"]
      restartPolicy: Never

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, deploying Flink on Kubernetes with an operator ensures robust management of stream processing jobs. For always-on pipelines, focus on high availability and fault tolerance, while for batch backfills, prioritize resource efficiency and job completion speed.

#### Common mistakes
include:

Not configuring proper horizontal pod autoscalers.
Overlooking checkpointing and savepoints for recovery.
Ignoring the importance of statefulset configuration for high availability.

#### Follow-up
questions might explore more advanced topics like customizing Flink configurations with operator parameters or integrating monitoring tools like Prometheus for real-time metrics.

