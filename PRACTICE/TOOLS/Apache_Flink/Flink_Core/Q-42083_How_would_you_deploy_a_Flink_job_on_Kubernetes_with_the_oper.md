---
type: "practice"
difficulty: "Deep"
---

# How would you deploy a Flink job on Kubernetes with the operator for an always-on ingestion pipeline, and what differs from a batch backfill run?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Zero-loss Job Upgrades](<../../../../TOOLS/Apache_Flink/Flink_Core/Zero-loss_Job_Upgrades.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Deploying a Flink job on Kubernetes using an operator involves setting up a Kubernetes cluster, configuring the Flink deployment with the necessary resources, and leveraging the operator to manage the lifecycle of the Flink job. The key differences between deploying for an always-on ingestion pipeline versus a batch backfill run lie in resource allocation, fault tolerance, and data processing semantics.

#### Triaging the Question

The question is asking about deploying Apache Flink jobs on Kubernetes using an operator to create an always-on ingestion pipeline. It also seeks to understand how this differs from deploying Flink for batch backfill runs. The interviewer likely wants insight into your understanding of Kubernetes, Flink’s architecture, and the nuances between stream processing and batch processing.

#### Assumptions
Kubernetes Cluster: You have a Kubernetes cluster set up with appropriate resources.
Operator Setup: An operator like Apache Flink Operator or Debezium Operator is available to manage the Flink deployment.
Flink Application: The Flink job has been written and tested locally.
Step-by-Step Solution
1. Setting Up Kubernetes Cluster

Ensure you have a Kubernetes cluster with sufficient resources allocated for your Flink jobs. Typically, this involves:

Ensuring you have the necessary roles and permissions to deploy pods.
Configuring network policies if required.
2. Installing the Operator

Install the operator on your Kubernetes cluster. For Apache Flink, use:

BASH
kubectl apply -f https://raw.githubusercontent.com/strimzi/strimzi-kafka-operator/master/samples/application-templates/flink/operator-only.yaml
3. Defining Flink Job Specifications

Define a YAML manifest for the Flink job that includes details such as:

The Docker image of your Flink application.
Resource requirements (CPU, memory).
Parallelism settings.

#### Example
flink-job.yaml:

YAML
apiVersion: flink.apache.org/v1beta2
kind: FlinkJob
metadata:
  name: always-on-ingestion
spec:
  taskManager:
    parallelism: 4
    resourceClaim:
      resources:
        requests:
          cpu: "1"
          memory: "512Mi"
        limits:
          cpu: "2"
          memory: "1Gi"
  job:
    jar: <path-to-jar>
    main-class: com.example.IngestionJob
    arguments:
      - arg1
      - arg2
4. Deployment

Deploy the Flink job using kubectl or a Kubernetes dashboard.

BASH
kubectl apply -f flink-job.yaml
5. Monitoring and Scaling

Monitor the Flink job's status with:

BASH
kubectl get flinkjobs

Scale the job if needed by modifying the parallelism in the YAML manifest.

Always-On Ingestion Pipeline vs Batch Backfill Run
1. Resource Allocation
Always-on: Typically requires more robust resource management and scaling capabilities due to continuous operation.
Batch Backfill: Requires careful tuning of resources based on expected load, but does not need as much dynamic scaling.
2. Fault Tolerance
Always-on: Uses Flink’s fault-tolerance mechanisms such as checkpointing and exactly-once processing guarantees.
Batch Backfill: May use simpler error handling and recovery strategies since the job is run once.
3. Data Processing Semantics
Always-on: Continuous data ingestion with low latency, ensuring real-time or near-real-time processing.
Batch Backfill: Process historical data in batch mode, which may have higher latency but requires less complex setup for initial deployment.

#### Closing Summary
and Common Mistakes

#### Closing Summary

Deploying an always-on ingestion pipeline with Flink on Kubernetes involves setting up a cluster, deploying the operator, defining job specifications, and managing resources. The key differences between this and batch backfill runs lie in resource allocation, fault tolerance, and data processing semantics.

#### Common Mistakes
Ignoring Resource Constraints: Not properly configuring resource limits can lead to out-of-memory errors or poor performance.
Underestimating Fault Tolerance: Failing to set up proper checkpointing mechanisms for always-on pipelines can result in data loss.
Misunderstanding Batch vs Stream Processing: Confusing the processing semantics, which could lead to incorrect job configuration.

**Code:**

```yaml
apiVersion: flink.apache.org/v1beta2
kind: FlinkJob
metadata:
  name: always-on-ingestion
spec:
  taskManager:
    parallelism: 4
    resourceClaim:
      resources:
        requests:
          cpu: "1"
          memory: "512Mi"
        limits:
          cpu: "2"
          memory: "1Gi"
  job:
    jar: file:///path/to/jar.jar
    main-class: com.example.IngestionJob
    arguments:
      - arg1
      - arg2
```

**Example input / output:**

```text
# Deploy Flink job
kubectl apply -f flink-job.yaml

# Check status of the job
kubectl get flinkjobs

# Scale the job if needed
kubectl edit flinkjob always-on-ingestion
```

**Flow:**

1. Set up Kubernetes Cluster
2. Install Operator
3. Define Flink Job YAML
4. Deploy Flink Job with kubectl
5. Maintain and Monitor

