---
type: "practice"
difficulty: "Deep"
---

# With the KubernetesExecutor how do you set per-task pod resource requests limits and node selectors and what happens when a pod is evicted?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Airflow Architecture](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Apache Airflow's KubernetesExecutor allows for fine-grained control over Kubernetes pods, including setting per-task pod resource requests and node selectors. Understanding how these configurations work and what happens when a pod is evicted requires delving into the mechanics of Kubernetes and Airflow’s interaction with it.

#### Triaging the Question

The question revolves around configuring KubernetesExecutor in Apache Airflow to control the resources allocated to individual tasks, specifically through resource requests and node selectors. It also asks about the behavior when a pod is evicted. The key aspects being probed are:

Resource Requests: Defining CPU and memory requirements for each task.
Node Selectors: Ensuring tasks run on specific nodes based on labels.
Pod Eviction: Understanding what happens if a pod cannot satisfy resource requests or other constraints.

#### First-Principles Solution
Setting Per-Task Pod Resource Requests

To set per-task pod resource requests, you can use the resources parameter in your Airflow task definition. This parameter is part of the Kubernetes pod specification and allows you to define CPU and memory limits for individual tasks. Here’s an example:

PYTHON
from airflow import DAG
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

with DAG(
    'example_dag',
    schedule_interval=None,
    start_date=datetime(2023, 1, 1),
) as dag:
    task = KubernetesPodOperator(
        namespace='default',
        image="ubuntu:latest",
        cmds=["bash", "-cx"],
        resources={
            "requests": {
                "cpu": "500m",
                "memory": "64Mi"
            },
            "limits": {
                "cpu": "1",
                "memory": "256Mi"
            }
        },
        task_id="example_task",
    )
Setting Node Selectors

Node selectors ensure that tasks run on specific nodes based on labels. You can specify these in the node_selector parameter of the KubernetesPodOperator.

PYTHON
from airflow import DAG
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

with DAG(
    'example_dag',
    schedule_interval=None,
    start_date=datetime(2023, 1, 1),
) as dag:
    task = KubernetesPodOperator(
        namespace='default',
        image="ubuntu:latest",
        cmds=["bash", "-cx"],
        node_selector={"example-node-label": "true"},
        task_id="example_task",
    )
What Happens When a Pod is Evicted

When a pod cannot satisfy its resource requests or other constraints, Kubernetes will evict the pod. This can happen for several reasons:

Resource Starvation: If the node does not have enough resources to meet the requests defined in the pod spec.
Node Maintenance: During maintenance events like updates or patching.
Node Failure: In case of a failure, Kubernetes may evict pods from that node.

When a pod is evicted:

The task will be rescheduled on another available node if the DAG and operator are configured to handle this scenario.
If no suitable nodes are available (based on node_selector), the task will remain in an error state until resources become available or the constraints change.

#### Closing Summary

In summary, configuring per-task pod resource requests and node selectors with KubernetesExecutor involves specifying resources and node_selector parameters. The behavior when a pod is evicted depends on Kubernetes’ policies and the availability of suitable nodes. It’s crucial to consider these factors in designing robust Airflow workflows that can handle dynamic environments.

#### Common Mistakes/Follow-Ups
Over-provisioning Resources: Setting overly high resource requests can lead to eviction issues.
Insufficient Node Selection: Failing to properly label and select nodes can result in tasks not running where needed.
Handling Evictions Gracefully: Designing the DAG to handle rescheduling after pod evictions ensures continuous operation.

**Comparison:**

```
Resource Type	Request	Limit
CPU	500m	1
Memory	64Mi	256Mi
```

**Code:**

```python
from airflow import DAG
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator

with DAG(
    'example_dag',
    schedule_interval=None,
    start_date=datetime(2023, 1, 1),
) as dag:
    task = KubernetesPodOperator(
        namespace='default',
        image="ubuntu:latest",
        cmds=["bash", "-cx"],
        resources={
            "requests": {
                "cpu": "500m",
                "memory": "64Mi"
            },
            "limits": {
                "cpu": "1",
                "memory": "256Mi"
            }
        },
        node_selector={"example-node-label": "true"},
        task_id="example_task",
    )
```

**Example input / output:**

```text
Input (Airflow Task Definition):
KubernetesPodOperator(
  namespace='default',
  image="ubuntu:latest",
  cmds=["bash", "-cx"],
  resources={
    "requests": {
      "cpu": "500m",
      "memory": "64Mi"
    },
    "limits": {
      "cpu": "1",
      "memory": "256Mi"
    }
  },
  node_selector={"example-node-label": "true"},
  task_id="example_task"
)

Output (Pod Spec):
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: ubuntu:latest
    command: ["bash", "-cx"]
    resources:
      requests:
        cpu: "500m"
        memory: "64Mi"
      limits:
        cpu: "1"
        memory: "256Mi"
    nodeSelector:
      example-node-label: "true"
```

**Data:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: ubuntu:latest
    command: ["bash", "-cx"]
    resources:
      requests:
        cpu: "500m"
        memory: "64Mi"
      limits:
        cpu: "1"
        memory: "256Mi"
    nodeSelector:
      example-node-label: "true"
```

**Flow:**

1. No
2. Yes
3. Pod Scheduling
4. Node with sufficient resourcesand matching labels
5. Is Pod Evicted?
6. Task Continues to Run onNode
7. Task Rescheduled on SuitableNode if Available, else TaskFails

