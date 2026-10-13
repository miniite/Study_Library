---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Airflow_on_Cloud"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Airflow on AWS Azure and GCP

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** On the big clouds you can run Airflow yourself on VMs/Kubernetes or consume a managed service (Amazon MWAA, Cloud Composer, Astronomer, Azure options) — the choice trades control and cost for operational effort.

### 1. The big picture
Airflow's components (scheduler, workers, webserver, database, DAG storage, log storage, secrets) map neatly onto cloud services: a managed relational database, object storage for DAGs and logs, a container/Kubernetes platform for workers, a secrets manager, and IAM for access. The book's cloud chapters (design strategies, AWS, Azure, GCP) show how to pick services and deploy, then use the platform's *native* operators to orchestrate cloud-specific work — for example a serverless movie-ranking pipeline on GCP, or a similar analytics pipeline on AWS or Azure.

The key design question is not "can I run Airflow in the cloud?" (yes) but **"who runs it, and what do I orchestrate?"**

### 2. Core concepts and vocabulary
- **Self-managed** — you deploy Airflow on VMs or Kubernetes (e.g., with the official Helm chart).
- **Managed Airflow** — vendor operates scheduler/webserver/DB for you: **Amazon MWAA**, **Google Cloud Composer**, **Astronomer (Astro)**, and Azure options (Data Factory managed Airflow, AKS deployments).
- **Provider packages** — per-cloud operator/hook libraries (`apache-airflow-providers-amazon`, `-google`, `-microsoft-azure`).
- **IAM/identity** — roles and service accounts that grant Airflow access to cloud services without static keys.
- **Object storage** — S3/GCS/Blob used for DAGs, logs, data staging.
- **Orchestrate, don't compute** — Airflow triggers serverless/managed compute (Glue, EMR, BigQuery, Dataflow, Synapse, Databricks) instead of doing heavy work itself.
- **Environment** — a managed Airflow deployment unit with its own DAG bucket, size class, Airflow version and plugins/requirements.
- **Worker autoscaling** — adding/removing workers based on queue depth.
- **Network integration** — VPC/VNet peering, private endpoints for DB and services.

### 3. How it works, step by step
**Choosing a deployment strategy**

1. Inventory needs: number of DAGs/tasks, team size, compliance, version-pinning needs, existing Kubernetes skills.
2. Decide *self-managed vs managed*:
   - Few platform engineers, standard needs → managed service.
   - Custom plugins, strict version control, hybrid/on-prem → self-managed on Kubernetes.
3. Provision infrastructure (IaC): network, metadata DB, DAG/log buckets, secrets, identities.
4. Deploy Airflow (Helm on Kubernetes or create a managed environment).
5. Configure connections through IAM roles/workload identity rather than keys.
6. Develop DAGs using provider operators that call cloud services.
7. Set up CI/CD that syncs DAGs to the bucket/git-sync and promotes through environments.

**Typical orchestration pattern (any cloud):** sensor waits for new data in object storage → operator launches a serverless transformation → operator loads to the warehouse → quality check → notification.

### 4. Worked example
A GCP serverless pattern (conceptual, from the book's movie-ranking use case): upload ratings to GCS, rank movies with a managed service, write result to BigQuery.
```python
from datetime import datetime
from airflow import DAG
from airflow.providers.google.cloud.transfers.local_to_gcs import LocalFilesystemToGCSOperator
from airflow.providers.google.cloud.operators.dataflow import DataflowTemplatedJobStartOperator
from airflow.providers.google.cloud.transfers.gcs_to_bigquery import GCSToBigQueryOperator

with DAG("movie_ranking_gcp", start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False) as dag:
    upload = LocalFilesystemToGCSOperator(
        task_id="upload_ratings", src="/data/ratings/{{ ds }}.json",
        dst="ratings/{{ ds }}.json", bucket="acme-movies-raw", gcp_conn_id="google_cloud_default")

    rank = DataflowTemplatedJobStartOperator(
        task_id="rank_movies", template="gs://acme-templates/rank_movies",
        parameters={"input": "gs://acme-movies-raw/ratings/{{ ds }}.json",
                    "output": "gs://acme-movies-curated/rankings/{{ ds }}/"},
        location="europe-west1")

    load = GCSToBigQueryOperator(
        task_id="load_rankings", bucket="acme-movies-curated",
        source_objects=["rankings/{{ ds }}/*.json"],
        destination_project_dataset_table="analytics.movie_rankings${{ ds_nodash }}",
        source_format="NEWLINE_DELIMITED_JSON", write_disposition="WRITE_TRUNCATE")

    upload >> rank >> load
```
`WRITE_TRUNCATE` onto a *partition decorator* makes the load idempotent. The AWS equivalent swaps in `S3Hook`/`GlueJobOperator`/`RedshiftDataOperator`; Azure uses `WasbHook`, Data Factory and Synapse operators.

Check the environment: the DAG bucket receives your file → the scheduler picks it up within a minute → the Graph view shows the three tasks; logs appear in the cloud logging service.

### 5. Under the hood   *(optional)*
- **Amazon MWAA:** runs Airflow on AWS-managed Fargate containers; DAGs, plugins and `requirements.txt` live in an S3 bucket; Celery executor behind the scenes with autoscaled workers; metadata DB hidden; execution role provides IAM permissions.
- **Cloud Composer:** Airflow on a GKE cluster (Composer 2 autoscaling), DAGs in a GCS bucket, Cloud SQL metadata DB; integrates with Cloud Logging/Monitoring and Workload Identity.
- **Astronomer:** vendor control plane with deployments on your cloud account; focus on Airflow-native CLI, CI/CD and observability.
- **Self-managed on Kubernetes:** Helm chart installs scheduler, webserver, triggerer, workers or KubernetesExecutor; DAGs via git-sync sidecar or baked image; PostgreSQL via managed service.
- **Identity:** prefer IAM roles for service accounts (EKS IRSA, GKE Workload Identity, Azure managed identity) so no long-lived keys sit in connections.
- **Version/dependency management:** managed services lag Airflow releases and constrain package installation; know upgrade policy.
- **Hybrid triggers:** Airflow can orchestrate across clouds via provider operators and generic HTTP/SSH operators.

### 6. Performance and design implications
- **Orchestration vs compute:** moving heavy transformation to Dataflow/EMR/BigQuery/Synapse keeps Airflow workers small and cheap.
- **Cost model:** managed services bill by environment size/hour even when idle; Kubernetes can scale to zero for workers but you pay ops time.
- **Network latency & egress:** keep Airflow, data and compute in one region; cross-region transfers are slow and billed.
- **Startup times:** serverless jobs (Glue/Dataflow) can take minutes to start — use deferrable operators for waiting.
- **Quotas:** API rate limits for cloud services; use pools and retries with backoff.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `helm install airflow apache-airflow/airflow` | Deploy on Kubernetes | Self-managed |
| Upload to DAG bucket (`aws s3 cp` / `gsutil cp`) | Deploy DAGs on managed service | MWAA / Composer |
| `requirements.txt` in environment | Install extra Python packages | Managed envs |
| `AWS_DEFAULT_REGION`, `aws_conn_id` | Region and connection | AWS operators |
| Workload Identity / IRSA | Keyless access | Kubernetes on cloud |
| Cloud Logging / CloudWatch | Central logs/metrics | Operations |
| Terraform/CDK | Provision environment | Repeatable infra |
| Deferrable operator variants | Async waiting | Long cloud jobs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Managed Airflow | Small team, standard needs, want SLAs and less patching |
| Self-managed Kubernetes | Need custom images/plugins, strict versions, multi-cloud |
| Single cloud-native services | You're all-in on one cloud and want tight integration |
| Airflow for orchestration only | Tasks are heavy compute — never run them on workers |
| Multiple small environments | Isolation needed; mind cost of idle environments |

### 9. Common mistakes
- **Running heavy transformations inside Airflow workers** → OOM, slowness → delegate to managed compute.
- **Storing cloud keys in connections** → leakage, rotation pain → IAM roles/workload identity.
- **Ignoring region/egress** → cost and latency → co-locate.
- **Assuming managed = no ops** → still own DAG quality, dependencies, cost → monitor and review.
- **Not versioning requirements** → environment breaks on update → pin versions/constraints.
- **Non-idempotent loads** → duplicates on retry → partition overwrite/MERGE.

### 10. 🎯 Interview kit
**30-second answer:** "In the cloud I'd choose between a managed Airflow (MWAA/Composer/Astronomer) and self-managed Kubernetes based on team skills and control needs, keep Airflow as an orchestrator that triggers cloud-native compute, and use IAM roles instead of static keys."

**Structure a longer answer:** clarify constraints → compare managed vs self-managed → architecture (storage, DB, identity, network) → trade-offs (cost, version lag) → example pipeline.

**Likely follow-ups:**
- *What does MWAA manage for you?* → Scheduler, workers, webserver, DB patching/scaling; you manage DAGs, requirements, IAM.
- *How do you authenticate to cloud services?* → Execution role/workload identity, not keys.
- *Why shouldn't Airflow do the transform?* → Not a data processing engine; scaling and memory.
- *How do you deploy DAGs?* → Sync to bucket/git-sync via CI.

**Signals interviewers look for:** knowing "orchestrate, don't compute"; identity and network awareness; cost consciousness.

### 11. Check yourself
- [ ] Choose managed vs self-managed for two scenarios and justify.
- [ ] Map Airflow components to services on one cloud of your choice.
- [ ] Explain how to authenticate without static credentials.
- [ ] Design a DAG that loads daily data to a warehouse idempotently.
- [ ] List three pitfalls of managed Airflow.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow in GCP
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow in the clouds
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow on Azure
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow on AWS
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Project Finding the fastest way to get around NYC › Project: Finding the fastest way to get around NYC
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Choosing Technologies Across the Data Engineering Lifecycle › Multicloud

