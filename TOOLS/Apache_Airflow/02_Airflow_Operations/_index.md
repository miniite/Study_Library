---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Airflow in Practice and Production

**L3 Practitioner** · [↑ Apache Airflow](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Operating Airflow well means getting work started the right way (triggers, sensors), shipping dependencies cleanly (containers, custom components), proving it works (tests, best practices), and running it safely at scale (production hardening, security, cloud deployment).

### 1. The big picture
Once DAGs work on a laptop, the questions change: *How do I wait for late data without wasting workers? How do I run a job needing different libraries? How do I know a change won't break production? Who may see credentials? Where does Airflow live — VM, Kubernetes, managed service?* The Operations lessons answer these.

| Lesson | Theme |
|---|---|
| **Triggering and Sensors** | Start/wait on events: sensors, deferrable, DAG-to-DAG, datasets |
| **Containerized Tasks** | Isolate dependencies with Docker/Kubernetes pods |
| **Custom Components** | Hooks, operators and sensors for in-house systems |
| **Best Practices** | Idempotent, atomic, parse-light, bounded DAGs |
| **Testing** | Integrity, unit, integration and whole-DAG tests |
| **Production and Security** | HA, RBAC, secrets, monitoring, deployments |
| **Airflow on Cloud** | Managed vs self-managed on AWS/GCP/Azure |

### 2. Core concepts and vocabulary
- **Sensor, poke vs reschedule vs deferrable** — three ways to wait.
- **KubernetesPodOperator / DockerOperator** — run a container per task.
- **Hook / operator / provider** — building blocks for integrations.
- **Idempotent, atomic, deterministic** — task design properties.
- **DagBag integrity test** — CI gate for importability.
- **Fernet key, secrets backend, RBAC** — security primitives.
- **Managed Airflow (MWAA, Composer, Astronomer)** vs **self-managed (Helm/Kubernetes)**.
- **Pool** — resource guard shared by tasks.

### 3. How it works, step by step
A maturity path for a team:

1. **Make it correct:** idempotent tasks, logical-date templating, retries and timeouts (Best Practices).
2. **Make it testable:** integrity + unit tests in CI (Testing).
3. **Make dependencies manageable:** package shared hooks/operators (Custom Components); isolate odd workloads in containers (Containerized Tasks).
4. **Make it event-aware:** sensors in reschedule/deferrable mode, datasets, triggers (Triggering and Sensors).
5. **Make it safe:** secrets backend, SSO/RBAC, remote logging, metrics and alerts (Production and Security).
6. **Make it run somewhere sustainable:** pick managed vs self-managed and orchestrate cloud compute (Airflow on Cloud).

### 4. Worked example
One DAG that touches most of the operations themes:
```python
with DAG("partner_feed", start_date=datetime(2024, 1, 1), schedule="@daily", catchup=False,
         default_args={"retries": 3, "retry_delay": timedelta(minutes=10),
                       "on_failure_callback": notify_failure}) as dag:

    wait = S3KeySensor(task_id="wait_feed", bucket_key="s3://partner/feed_{{ ds_nodash }}.csv",
                       mode="reschedule", poke_interval=600, timeout=8 * 3600, pool="sensor_pool")

    transform = KubernetesPodOperator(task_id="transform", image="registry/feed-transform:1.3.0",
                                      arguments=["--day", "{{ ds }}"], is_delete_operator_pod=True)

    load = SnowflakeOperator(task_id="load", sql="CALL load_feed('{{ ds }}')",   # idempotent proc
                             snowflake_conn_id="warehouse")                     # connection from secrets backend

    wait >> transform >> load
```
- Sensor releases its slot (`reschedule`) and is capped by a pool.
- Transformation runs in its own image; the DAG stays thin.
- Credentials come from the secrets backend; alerts via callback.
- A CI integrity test guards the file; unit tests cover `feed-transform`.

### 5. Under the hood   *(optional)*
- **Resource plumbing:** slots → pools → executor parallelism → worker capacity; sensors in poke mode consume slots for their entire wait.
- **Isolation spectrum:** `PythonOperator` (shared env) → `PythonVirtualenvOperator` → container operators → external compute services.
- **Security layers:** transport (TLS), authentication (SSO), authorization (RBAC), secrets (backend + Fernet), execution (service accounts/namespaces).
- **Failure model:** at-least-once execution; correctness comes from idempotent sinks, not from "exactly-once" scheduling.

### 6. Performance and design implications
- Parse time, DB size and queue depth are the three health indicators of a busy deployment.
- Containerization trades latency for isolation; avoid it for tiny frequent tasks.
- Over-eager sensors and aggressive pokes create DB and slot pressure.
- Cost levers: right-sized environments, autoscaling workers, off-peak backfills.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `mode="reschedule"`, `deferrable=True` | Release slots while waiting | Long sensors |
| `KubernetesPodOperator(image=…)` | Container task | Dependency isolation |
| `pytest` + `DagBag` | CI integrity tests | Every PR |
| `secrets.backend`, `rotate-fernet-key` | Credential management | Production |
| `airflow db clean` | DB housekeeping | Scheduled maintenance |
| `pool`, `max_active_runs` | Throttle | Backfills, APIs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Managed Airflow | Small platform team, standard needs |
| Self-managed Kubernetes | Custom plugins/versions, hybrid cloud |
| Containers per task | Conflicting dependencies, polyglot tasks |
| Datasets/triggers | Cross-team DAG dependencies |
| Heavy unit tests | Pure glue DAGs — integrity tests suffice |

### 9. Common mistakes
- Poke-mode sensors everywhere → slot starvation.
- Latest tags and baked-in secrets → irreproducible and insecure.
- Skipping integrity tests → import errors in production.
- Shared mutable local disks → failures after worker changes.
- Treating Airflow as compute → OOM workers.

### 10. 🎯 Interview kit
**30-second answer:** "Operating Airflow is about robust triggering and waiting, isolated and reusable task code, layered testing, and a hardened, monitored deployment — managed or on Kubernetes — while keeping Airflow an orchestrator rather than a compute engine."

**Structure a longer answer:** reliability → isolation → testing → security → platform choice, each with one trade-off and one real incident.

**Likely follow-ups:** *Poke vs deferrable?* *KPO vs executor?* *How do you test DAGs?* *Where do secrets live?* *Managed vs self-managed?*

**Signals interviewers look for:** end-to-end operational thinking, trade-off fluency, security and cost awareness.

### 11. Check yourself
- [ ] Propose a deployment architecture for 200 DAGs and 3 teams.
- [ ] Choose a waiting strategy for a late partner file and justify.
- [ ] Describe the CI pipeline you'd set up for DAG code.
- [ ] Explain how to isolate a team's conflicting Python dependencies.
- [ ] List the security controls you'd enable before go-live.
<!-- explainer:end -->

## In this section (study in order)

1. [Triggering Sensors and External Systems](<Triggering_and_Sensors/_index.md>) (L4 Advanced)
2. [Custom Operators Hooks and Sensors](<Custom_Components/_index.md>) (L4 Advanced)
3. [Testing Airflow DAGs](<Testing/_index.md>) (L4 Advanced)
4. [Running Tasks in Containers](<Containerized_Tasks/_index.md>) (L4 Advanced)
5. [Operating and Securing Airflow](<Production_and_Security/_index.md>) (L4 Advanced)
6. [Airflow Best Practices](<Best_Practices/_index.md>) (L4 Advanced)
7. [Airflow on AWS Azure and GCP](<Airflow_on_Cloud/_index.md>) (L4 Advanced)

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Testing
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Building custom components
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running tasks in containers › Airflow in practice
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — In the clouds
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 31
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Operating Airflow in production
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Triggering workflows
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — preface

