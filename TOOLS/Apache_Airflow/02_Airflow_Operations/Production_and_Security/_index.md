---
id: "TOOLS/Apache_Airflow/02_Airflow_Operations/Production_and_Security"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Operating and Securing Airflow

**L4 Advanced** · [↑ Airflow in Practice and Production](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Running Airflow in production means a robust metadata DB and executor, monitoring and alerting, controlled access (RBAC, secrets backends, encryption), and disciplined deployment of DAGs and configuration.

### 1. The big picture
A laptop `airflow standalone` and a production platform differ in four ways: **reliability** (HA components, backups, upgrades), **observability** (metrics, logs, alerts for failures and lateness), **security** (who may see/trigger what, where credentials live) and **change management** (how code reaches the DAGs folder, how environments are separated). The book's "Maintaining Airflow in production" and "Securing Airflow" chapters cover these themes; this lesson also brings in operational knowledge from running Airflow at scale.

### 2. Core concepts and vocabulary
- **Metadata database** — PostgreSQL/MySQL holding all state; needs backups, pooling and maintenance.
- **RBAC** — role-based access control in the webserver: Admin, User, Op, Viewer, Public + custom roles, with DAG-level permissions.
- **Connection / Variable** — stored credentials/config; encrypted with a **Fernet key**.
- **Secrets backend** — pluggable provider for connections/variables from Vault, AWS Secrets Manager, GCP Secret Manager, Azure Key Vault.
- **Authentication** — password, LDAP, OAuth/OIDC, SSO via the Flask-AppBuilder auth manager.
- **Webserver secret key** — signs sessions; must be the same across webserver replicas.
- **StatsD / OpenTelemetry metrics** — counters/timers for scheduler, executor, DAG runs.
- **SLA / callbacks** — `on_failure_callback`, `sla_miss_callback`, email/Slack alerts.
- **Remote logging** — shipping task logs to S3/GCS/Azure Blob/Elastic.
- **Airflow ENV configuration** — `AIRFLOW__SECTION__KEY` environment variables override `airflow.cfg`.

### 3. How it works, step by step
A pragmatic hardening sequence:

1. **Infrastructure:** managed PostgreSQL with backups; Celery/Kubernetes executor; at least two schedulers and two webservers behind a load balancer; shared or git-synced DAGs.
2. **Configuration as code:** set values via environment/Helm values, not by hand-editing files; keep `fernet_key`, `webserver.secret_key`, DB URI in a secrets store.
3. **Identity:** enable SSO/LDAP; map groups to roles; remove default admin accounts; limit the "Public" role.
4. **Secrets:** turn on a secrets backend so connections/variables are *not* stored in the metadata DB; rotate Fernet keys when needed (`airflow rotate-fernet-key`).
5. **Network:** TLS on the webserver, private networking for DB/broker, firewall workers' egress.
6. **Logging & monitoring:** remote logs; StatsD → Prometheus/Grafana; alerts on scheduler heartbeat, queue depth, DAG import errors, SLA misses, DB connections.
7. **Deployment:** CI runs lint + integrity tests, builds image, rolls out; DAG changes via git-sync; blue/green or rolling upgrades with DB migration (`airflow db migrate`).
8. **Housekeeping:** `airflow db clean`, log retention, pool/concurrency review.

### 4. Worked example
Environment-based configuration sketch (Kubernetes/Docker env):
```bash
# core
AIRFLOW__CORE__EXECUTOR=CeleryExecutor
AIRFLOW__CORE__FERNET_KEY=<from secret store>
AIRFLOW__DATABASE__SQL_ALCHEMY_CONN=postgresql+psycopg2://airflow:***@pg-primary:5432/airflow

# secrets backend: connections and variables come from AWS Secrets Manager
AIRFLOW__SECRETS__BACKEND=airflow.providers.amazon.aws.secrets.secrets_manager.SecretsManagerBackend
AIRFLOW__SECRETS__BACKEND_KWARGS='{"connections_prefix":"airflow/connections","variables_prefix":"airflow/variables"}'

# logging & metrics
AIRFLOW__LOGGING__REMOTE_LOGGING=True
AIRFLOW__LOGGING__REMOTE_BASE_LOG_FOLDER=s3://acme-airflow-logs/prod
AIRFLOW__METRICS__STATSD_ON=True
AIRFLOW__METRICS__STATSD_HOST=statsd-exporter

# webserver
AIRFLOW__WEBSERVER__SECRET_KEY=<from secret store>
AIRFLOW__WEBSERVER__EXPOSE_CONFIG=False
```
Alerting at DAG level:
```python
def notify_failure(context):
    ti = context["task_instance"]
    send_to_slack(f":red_circle: {ti.dag_id}.{ti.task_id} failed for {context['ds']} — {ti.log_url}")

default_args = {"retries": 2, "on_failure_callback": notify_failure,
                "sla": timedelta(hours=2), "execution_timeout": timedelta(hours=1)}
```
Verify: a connection fetched from the secrets backend appears in tasks but *not* in the UI's connection list; killing one scheduler pod does not stop scheduling.

### 5. Under the hood   *(optional)*
- **Fernet:** symmetric encryption of connection passwords and sensitive variables in the DB. If the key is lost, those fields cannot be decrypted; rotation supports multiple keys (comma-separated).
- **RBAC model:** permissions are (action, resource) pairs such as `can_read on DAG:sales`; DAG-level access is configured by `access_control` in the DAG or per-role.
- **Secrets lookup order:** environment variables → secrets backend → metadata DB; the first hit wins.
- **Scheduler HA:** schedulers coordinate via row-level locks on the DB; they don't need leader election, but DB latency matters.
- **Task-level isolation:** workers run user code with the worker's privileges — a malicious DAG author can reach anything the worker can; separate teams with separate deployments/Kubernetes namespaces and service accounts where trust is low.
- **Metrics of interest:** `scheduler.heartbeat`, `executor.queued_tasks`, `dag_processing.total_parse_time`, `dagrun.duration.*`, `pool.starving_tasks`, `ti.failures`.
- **Upgrades:** run `airflow db migrate` with the new version, test with a DB copy; provider package versions are upgraded independently.

### 6. Performance and design implications
- **DB sizing & hygiene:** connection pooling (PgBouncer), regular `db clean`, indexes managed by migrations; slow UI usually means a bloated DB.
- **Parse capacity:** scale `parsing_processes` and keep DAGs light.
- **Alert quality:** alert on *symptoms* (late data, failed critical DAGs), not each task retry.
- **Cost:** autoscale workers/pods; right-size pools and concurrency to avoid idle fleets.
- **Isolation vs. simplicity:** one shared Airflow is cheap but a noisy-neighbour risk; multi-tenant setups need quotas.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `airflow db migrate`, `airflow db check` | Upgrade/verify DB | Deployments |
| `airflow db clean --clean-before-timestamp …` | Purge old records | Housekeeping |
| `airflow rotate-fernet-key` | Re-encrypt secrets | Key rotation |
| `airflow users create` | Create accounts | Bootstrap (non-SSO) |
| `secrets.backend` | Use external secret store | Production credentials |
| `logging.remote_logging` | Ship logs | Ephemeral workers/pods |
| `webserver.expose_config=False` | Hide config in UI | Security |
| `on_failure_callback`, `sla` | Alerting | Operational awareness |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Managed Airflow (MWAA/Composer/Astronomer) | You want less ops and accept provider limits |
| Self-managed on Kubernetes | You need full control and have platform engineers |
| Secrets backend | Compliance requires central rotation/audit |
| Metadata-DB-stored connections | Small team, low risk — simpler |
| One shared instance | Teams trust each other and share standards |
| Instance per team/env | Strong isolation, differing Airflow versions needed |

### 9. Common mistakes
- **Default/dev settings in production** (SQLite, Sequential executor, example DAGs) → instability → use PostgreSQL, disable examples.
- **Fernet key not set or lost** → plaintext/undecryptable secrets → manage via secret store with backups.
- **Everyone is Admin** → accidental or malicious changes → least-privilege roles.
- **Logs only on local disks** → lost with pods → remote logging.
- **No DB maintenance** → UI crawls → scheduled cleanup.
- **Alert fatigue** → ignored pages → alert on critical outcomes with runbook links.
- **Deploying DAGs by copying into running workers** → version skew → immutable images/git-sync with atomic switches.

### 10. 🎯 Interview kit
**30-second answer:** "For production I'd run Airflow with a managed Postgres, a scalable executor, HA schedulers, remote logging and metrics; put credentials in a secrets backend with a Fernet key; use SSO and RBAC; and deploy via CI with integrity tests and immutable images."

**Structure a longer answer:** clarify scale/compliance → reliability (HA, DB) → security (RBAC, secrets, network) → observability (metrics/alerts) → change management and cost.

**Likely follow-ups:**
- *Where are credentials stored?* → Encrypted in the metadata DB with Fernet, or in an external secrets backend.
- *How do you detect a stuck scheduler?* → Heartbeat metric/health endpoint, DAG runs not created, queue depth.
- *How do you restrict a team to its DAGs?* → RBAC roles with DAG-level permissions, or separate deployments.
- *How do you upgrade safely?* → Staging, DB backup, `db migrate`, rolling deploy.

**Signals interviewers look for:** thinking beyond "it runs" — HA, security, observability, cost, upgrade paths.

### 11. Check yourself
- [ ] List five production hardening steps in order of priority.
- [ ] Explain Fernet keys and what happens on rotation.
- [ ] Describe how you would give team A access only to its DAGs.
- [ ] Name four metrics/alerts you would configure.
- [ ] Outline a safe upgrade procedure.
<!-- explainer:end -->

## References

- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Operating Airflow in production
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Securing Airflow
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow in GCP
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow on Azure
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Running code samples › Chapter 33
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Airflow on AWS
- *Hadoop: The Definitive Guide* — Tom White — Setting Up a Hadoop Cluster › An example

