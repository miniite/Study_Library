---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Containerized_Tasks"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Running Tasks in Containers

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Containerized tasks run each task's code inside its own Docker image (via `DockerOperator` or `KubernetesPodOperator`), removing dependency conflicts between pipelines and keeping Airflow itself thin.

### 1. The big picture
As a team grows, every pipeline wants different Python packages, system libraries and even languages. Installing all of them on shared Airflow workers causes version conflicts ("pandas 1.x vs 2.x"), large images, security worries and fragile upgrades. Airflow's answer is to treat Airflow as the *orchestrator* only: each task starts a **container** from an image that carries its own dependencies, runs a command, exits, and Airflow reads the exit code (and logs) to set the task state.

Benefits: isolation, reproducibility (same image in dev and prod), polyglot tasks, per-task resource requests, and independent release cycles for task code vs DAG code. Cost: image build/registry discipline, container start-up latency and more moving parts.

### 2. Core concepts and vocabulary
- **Container image** — immutable package with code + dependencies, tagged and stored in a registry.
- **DockerOperator** — runs a container on a Docker daemon reachable from the worker.
- **KubernetesPodOperator (KPO)** — creates a pod on a Kubernetes cluster, waits, streams logs, cleans up.
- **KubernetesExecutor** — a *different* idea: runs *every* Airflow task in a pod. KPO runs only specific tasks in custom images, even with other executors.
- **Image pull policy** — `Always`, `IfNotPresent`; matters for mutable tags like `latest`.
- **Resource requests/limits** — CPU/memory reservations in pods.
- **Environment variables / secrets** — how config and credentials reach the container.
- **Volume mounts** — shared storage between pods or from the node.
- **Entrypoint/command** — what the container runs; exit code ≠ 0 means failure.
- **XCom from container** — written to `/airflow/xcom/return.json` when `do_xcom_push=True`.

### 3. How it works, step by step
1. Task code is packaged as an image with a CLI entrypoint, e.g. `fetch-ratings --start 2024-01-01 --end 2024-01-02 --output s3://…`.
2. The DAG declares an operator with `image`, `command`/`arguments`, env vars and (templated) parameters.
3. When scheduled, the worker (or executor) creates the container/pod on the target runtime.
4. The runtime pulls the image (if needed), starts the container; stdout/stderr stream to the Airflow task log.
5. Airflow waits for exit; exit code 0 → `success`, otherwise `failed` (and retries apply).
6. Optional: results come back through the XCom sidecar file or through external storage the container wrote to.
7. The container/pod is removed (`auto_remove`/`is_delete_operator_pod`).

### 4. Worked example
```python
from datetime import datetime
from airflow import DAG
from airflow.providers.cncf.kubernetes.operators.pod import KubernetesPodOperator
from airflow.providers.docker.operators.docker import DockerOperator
from kubernetes.client import models as k8s

with DAG("ratings_pipeline", start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False) as dag:

    # Local/dev: Docker daemon on the worker
    fetch = DockerOperator(
        task_id="fetch_ratings",
        image="registry.example.com/ratings-fetch:1.4.2",       # pinned tag, never :latest
        command=["fetch-ratings", "--start", "{{ data_interval_start | ds }}",
                 "--end", "{{ data_interval_end | ds }}", "--output", "/data/{{ ds }}.json"],
        mounts=[],  # e.g. docker.types.Mount(source="/var/data", target="/data", type="bind")
        auto_remove="success",
        network_mode="bridge",
        environment={"API_HOST": "http://ratings-api:5000"},
    )

    # Production: pod on Kubernetes with explicit resources
    rank = KubernetesPodOperator(
        task_id="rank_movies",
        name="rank-movies",
        namespace="airflow-jobs",
        image="registry.example.com/movie-ranker:2.0.1",
        cmds=["rank-movies"],
        arguments=["--date", "{{ ds }}", "--input", "s3://lake/ratings/{{ ds }}/"],
        container_resources=k8s.V1ResourceRequirements(
            requests={"cpu": "500m", "memory": "1Gi"}, limits={"memory": "2Gi"}),
        env_vars={"LOG_LEVEL": "INFO"},
        get_logs=True,
        is_delete_operator_pod=True,
    )

    fetch >> rank
```
Reading results:
- Task logs show the container's stdout; a non-zero exit shows the pod's termination reason (e.g., `OOMKilled`).
- `kubectl get pods -n airflow-jobs` while running shows `Pending → Running → Completed`.
- If the task sits in `queued` but the pod is `ImagePullBackOff`, the registry credentials or tag are wrong.

### 5. Under the hood   *(optional)*
- **KPO flow:** builds a `V1Pod` spec from arguments → submits via the Kubernetes API → watches events/logs → maps final phase to task state → optionally deletes pod. A *base container* plus optional XCom sidecar share an `emptyDir` volume.
- **DockerOperator** talks to `docker_url` (unix socket or TCP) using the Docker SDK; the worker must have access to the socket (a security consideration).
- **Retry semantics:** each try creates a fresh container with a unique name suffix; idempotency is still required because side effects outside the container persist.
- **Secrets:** use Kubernetes Secrets mounted as env/volumes (`k8s.V1EnvVarSource`) or `airflow.providers.cncf.kubernetes.secret.Secret`; never bake secrets into images.
- **Image strategy:** a shared base image (OS, Python, common libs) + small layers per pipeline reduces pull times; node-level image caching helps.
- **Pod templates:** a cluster-wide default pod spec (service accounts, tolerations, node selectors) can be supplied and overridden per task.

### 6. Performance and design implications
- **Start-up latency** (image pull + pod scheduling: seconds to minutes) makes containers unsuitable for sub-minute tasks in tight chains.
- **Right-size resources** to avoid OOM kills or cluster waste; set both requests and limits.
- **Cache images** on nodes (smaller layers, pinned tags, `IfNotPresent`).
- **Keep data off the container filesystem**; use object stores/volumes so retries on another node still work.
- **Observability:** forward container logs to the Airflow log backend; export metrics from jobs separately.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DockerOperator(image=…, command=…)` | Run a container on Docker | Dev, simple VM deployments |
| `KubernetesPodOperator` | Run a pod | Production on Kubernetes |
| `container_resources=` | CPU/memory requests/limits | Prevent OOM and noisy neighbours |
| `is_delete_operator_pod=True` | Remove pod after run | Avoid pod build-up |
| `do_xcom_push=True` | Collect `/airflow/xcom/return.json` | Return small results |
| `image_pull_policy` | Control pulls | Pinned vs mutable tags |
| `kubectl logs/describe pod` | Inspect failures | Pending/crash debugging |
| `docker build -t reg/name:tag . && docker push` | Publish images | CI pipeline step |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Containerized tasks | Teams need conflicting dependencies or non-Python tools |
| `PythonVirtualenvOperator` | Light isolation for a few pure-Python deps without container infra |
| KubernetesPodOperator | You already operate Kubernetes and want per-task resources |
| KubernetesExecutor | You want *all* tasks isolated with the same Airflow image |
| Plain PythonOperator | Tasks are tiny and share dependencies |
| External services (Spark, ECS, Batch) | Compute needs exceed what a pod should do inside the cluster |

### 9. Common mistakes
- **Using `:latest` tags** → non-reproducible runs, stale caches → immutable versioned tags.
- **Putting secrets in image or DAG** → leaks → Secrets/connection backends.
- **No resource limits** → OOM or noisy neighbour → set requests and limits.
- **Writing results to container disk** → lost when pod exits → durable storage.
- **Not cleaning up pods** → cluster clutter, quota exhausted → delete on completion, TTL.
- **Mounting the Docker socket carelessly** → root-equivalent access → restrict or prefer Kubernetes.

### 10. 🎯 Interview kit
**30-second answer:** "I package each pipeline's code as an image and run it with KubernetesPodOperator or DockerOperator, so dependencies are isolated and Airflow stays a thin orchestrator. Tasks take parameters via templated CLI args and exchange data through external storage."

**Structure a longer answer:** clarify the dependency/isolation problem → mechanism (image, entrypoint, exit code, logs) → trade-offs (start-up latency, image management) → scenario (data-science team with conflicting libs).

**Likely follow-ups:**
- *KPO vs KubernetesExecutor?* → KPO = per-task custom image; executor = every task in a pod from the Airflow image.
- *How does a container return data?* → Small JSON via XCom sidecar; large data via storage.
- *How do you handle secrets?* → Kubernetes Secrets/secret backends, not images.
- *Why not install everything on workers?* → Conflicts, bloat, slow upgrades.

**Signals interviewers look for:** isolation reasoning, idempotency still required, resource management, security awareness.

### 11. Check yourself
- [ ] Write a KPO that runs `python -m job --date {{ ds }}` with 1 CPU/2 GiB.
- [ ] Explain KPO vs KubernetesExecutor.
- [ ] List causes of a pod stuck in `Pending` and `ImagePullBackOff`.
- [ ] Describe how to return a small result from a container.
- [ ] Say why pinned image tags matter for backfills.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running tasks in containers
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 33
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Containers
- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › Packaging dependencies
- *Hadoop: The Definitive Guide* — Tom White — N › P
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Communicating with external systems

