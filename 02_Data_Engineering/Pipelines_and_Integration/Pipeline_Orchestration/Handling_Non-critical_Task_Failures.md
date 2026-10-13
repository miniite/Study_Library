---
id: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures"
level: 4
type: "concept"
topic: "02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration"
tools: ["Apache_Airflow", "Apache_Spark"]
---

# Handling Non-critical Task Failures

**L4 Advanced** · [↑ Pipeline Orchestration](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Not every failure should stop a pipeline — classify tasks as critical or non-critical, retry transient errors, let optional steps fail without blocking downstream work, and always make the failure visible with alerts so "continue" never becomes "silently wrong".

### 1. The big picture
A pipeline is a chain of dependencies, and by default one failed task blocks everything after it. That is the right behavior for a task whose output is required for correctness (loading the fact table). It is the wrong behavior for a task whose output is nice-to-have (a notification, a secondary report, a cache warm-up, an enrichment from a flaky third-party API). Stopping the daily revenue load because the Slack post failed trades business value for nothing.

The pipeline books describe this as the decision of **when to halt and when to warn and continue**: validation and failure handling should distinguish severity. A good design has three ingredients: *retries* for transient problems, *explicit tolerance* for tasks that may fail, and *alerting/observability* so tolerated failures are still noticed and not confused with success.

### 2. Core concepts and vocabulary
- **Critical task** — its failure invalidates downstream results; the run must stop.
- **Non-critical (optional) task** — enrichment, notifications, auxiliary outputs; failure should not block the main path.
- **Transient vs deterministic failure** — a network blip (retry helps) vs a schema mismatch (retry just repeats it).
- **Retry / retry delay / backoff** — number of automatic re-attempts and the wait between them.
- **Task states** — e.g. in Airflow: `failed`, `up_for_retry`, `skipped`, `upstream_failed`, `success`.
- **Trigger rule** — which upstream outcomes allow a task to run (`all_success`, `all_done`, `one_failed`, `none_failed`…).
- **Fail loudly vs fail silently** — surfacing errors immediately vs swallowing them.
- **Dead-letter / quarantine** — place bad records aside with the reason instead of failing the whole batch.
- **Graceful degradation** — deliver a partial or older result with a clear marker rather than nothing.

### 3. How it works, step by step
1. **Classify each task** by blast radius: what breaks if it fails or is skipped? Mark it critical, optional, or "required for some consumers".
2. **Set retries by failure type.** Transient-prone tasks (API calls, network copies) get `retries=2–3` with a delay and backoff; deterministic-failure tasks get none or one.
3. **Decide what a final failure does.**
   - Critical: fail the task, block downstream, page someone.
   - Optional: allow the run to proceed — via a task that is not upstream of critical work, a downstream trigger rule that tolerates the failure (`all_done`/`none_failed`), or an in-code catch that logs and returns a "degraded" status.
4. **Make the failure visible.** Send an alert (email/Slack/pager) from a failure callback; record a metric; mark the run with a warning.
5. **Handle bad data separately.** Row-level problems go to quarantine with counts; the run fails only if the bad fraction exceeds a threshold.
6. **Ensure idempotency**, so a retried or re-run optional task cannot corrupt state.
7. **Review periodically.** A non-critical task that fails every day is a defect, not a feature.

### 4. Worked example
Airflow DAG: load orders (critical), enrich with weather (optional), notify.

```python
from airflow.utils.trigger_rule import TriggerRule

load_orders = PythonOperator(task_id="load_orders", python_callable=load,
                             retries=3, retry_delay=timedelta(minutes=5),
                             retry_exponential_backoff=True)

enrich_weather = PythonOperator(task_id="enrich_weather", python_callable=enrich,
                                retries=1,
                                on_failure_callback=slack_alert)   # visible, but...

build_report = PythonOperator(task_id="build_report", python_callable=report,
                              trigger_rule=TriggerRule.NONE_FAILED_MIN_ONE_SUCCESS)

load_orders >> [enrich_weather, build_report]
enrich_weather >> build_report
```
What to look at: if `enrich_weather` exhausts its retry and fails, the Slack callback fires, the UI shows it red, and `build_report` can still run because its trigger rule tolerates an upstream failure and the report code falls back to "no weather column". If `load_orders` fails, `build_report` is `upstream_failed` and nothing misleading is published. A task seen as `up_for_retry` is in its delay window — check the task log for the transient error and confirm the retry counter; `skipped` usually comes from branching/short-circuit logic or a sensor timeout configured with `soft_fail`.

### 5. Under the hood   *(optional at L4)*
- **Retries are per task instance:** the orchestrator re-queues after `retry_delay`; exponential backoff spreads load on a struggling dependency.
- **Trigger rules evaluate upstream states** at scheduling time; `all_done` ignores success/failure and runs after everything finishes (useful for cleanup), `none_failed` allows success or skipped.
- **Error vs failure:** a task *errors* when code raises; whether that becomes a *pipeline failure* is a policy decision (retry, ignore, escalate). Keeping the two separate makes handling explicit.
- **Silent degradation risk:** tolerating a failure changes the data contract — downstream consumers may receive output without the enrichment. Surface that via flags, row-level indicators, or quality metrics.
- **Cron-based fixed schedules** with variable upstream arrival turn "data not there yet" into recurring false failures; sensors, dataset triggers or retries with sensible windows address it.

### 6. Performance and design implications
- Retries add latency and load; cap attempts and use backoff so a downed API is not hammered.
- Long chains of optional tasks may hide real problems; track their success rate as a metric.
- Alert routing matters: critical failures page, optional failures go to a channel/ticket.
- Idempotent tasks make aggressive retry safe; non-idempotent ones need extra caution.
- Log attempt number, error, and run date to support debugging and audit.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `retries`, `retry_delay`, `retry_exponential_backoff` | Auto-retry configuration | Transient failures |
| `trigger_rule` (`all_done`, `none_failed`, `one_failed`…) | Controls whether a task runs after upstream failures | Optional tasks, cleanup, alerts |
| `on_failure_callback` / `on_retry_callback` | Hook to alert or log | Visibility for every failure |
| `execution_timeout`, `sla` | Bound runtime and detect lateness | Hung or slow tasks |
| `soft_fail` on sensors | Mark skipped instead of failed on timeout | Optional upstream data |
| Quarantine/dead-letter tables | Park bad rows | Row-level errors |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Fail the run — output required for correctness | Failure is in an optional extra |
| Warn and continue — enrichment, notifications | Downstream consumers assume the enrichment exists |
| Retry — transient, idempotent task | Deterministic errors; delays the alert |
| Quarantine rows — few bad records | Bad fraction large; indicates upstream breakage |
| Tolerate and flag degraded output | Consumers cannot distinguish degraded from complete |

### 9. Common mistakes
- Catching all exceptions and returning success → silent failure → log, alert, and set an explicit status.
- Making everything critical → one flaky notification blocks revenue reports → classify tasks.
- Making everything optional → bad data published → keep correctness-critical steps blocking.
- Retrying deterministic failures forever → noise and delay → fail fast after cause is clear.
- No alert on tolerated failures → problems accumulate unseen → alert and track rates.
- Confusing `skipped` with success → missing steps unnoticed → check why skipping occurred.

### 10. 🎯 Interview kit
**30-second answer:** I classify tasks by blast radius. Transient failures get a few retries with backoff; critical task failures stop the run and page; non-critical tasks are allowed to fail through trigger rules or caught errors but always raise an alert, so the pipeline continues without silently producing wrong data. Bad rows are quarantined and thresholds decide when to halt.

**Structure a longer answer:** clarify what each task feeds → retry policy → failure policy per class → visibility (alerts, metrics) → downstream contract for degraded output → idempotency.

**Likely follow-ups:**
- *Fail loudly vs silently?* → loud surfaces issues immediately; silent hides them until consumers complain.
- *Why a task is `up_for_retry`?* → it failed and is waiting `retry_delay`; read the log for the error.
- *What does `skipped` mean?* → upstream branching/short-circuit or soft-failed sensor.
- *Cron schedule with variable arrival?* → false failures; use sensors/event triggers or flexible retry windows.

**Signals interviewers look for:** nuanced severity handling, idempotency awareness, and operational visibility.

### 11. Check yourself
- [ ] Classify the tasks of a pipeline you know as critical or optional and justify.
- [ ] Choose a trigger rule for a report task that depends on an optional enrichment.
- [ ] Explain why alerting on tolerated failures still matters.
- [ ] Describe how you would stop retries from hammering a failing API.
<!-- explainer:end -->

## Implementations (tools)

- [Apache Airflow](<../../../TOOLS/Apache_Airflow/_index.md>)
- [Apache Spark](<../../../TOOLS/Apache_Spark/_index.md>)

## Practice questions

- [Basic] [What are the common task states in Airflow (like queued running success failed) and what does each mean?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13351_What_are_the_common_task_states_in_Airflow_like_queued_runni.md>)
- [Basic] [What is the difference between a warning and an error in your pipeline logs and how would you treat them differently?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13435_What_is_the_difference_between_a_warning_and_an_error_in_you.md>)
- [Basic] [What information would you look at in the Airflow logs when a task fails and you don't know why?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13572_What_information_would_you_look_at_in_the_Airflow_logs_when_.md>)
- [Basic] [What is the difference between a warning and an error in a pipeline and should a warning stop the run?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13636_What_is_the_difference_between_a_warning_and_an_error_in_a_p.md>)
- [Intermediate] [How would you make an Airflow task send an alert if the daily metrics pipeline fails?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13400_How_would_you_make_an_Airflow_task_send_an_alert_if_the_dail.md>)
- [Intermediate] [How would you handle an API that occasionally times out so your extract task doesn't fail permanently?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13627_How_would_you_handle_an_API_that_occasionally_times_out_so_y.md>)
- [Intermediate] [How do you decide the right retry count and exponential backoff for a task calling a flaky third-party API without masking a real persistent failure?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13700_How_do_you_decide_the_right_retry_count_and_exponential_back.md>)
- [Deep] [How would you set up retries and SLAs in Airflow so a transient Prometheus outage doesn't cascade into a missed daily anomaly-baseline refresh?](<../../../PRACTICE/02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Q-13490_How_would_you_set_up_retries_and_SLAs_in_Airflow_so_a_transi.md>)

## Related concepts

- [Scheduling Retries and Trigger Rules](<../../../TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules.md>) *(Airflow Core)*
- [Pipeline Design Patterns](<../Pipeline_Design/Pipeline_Design_Patterns.md>) *(Pipeline Design)*

## References

- *Ultimate Data Engineering Design Patterns* — Bragadeesh Sundararajan — Operationalizing Data Pipelines › 14. Operationalizing Data Pipelines
- *Data Engineering Fundamentals* — Zhaolong Liu — CHAPTER 5Data Orchestration and Workflows › CHAPTER 5Data Orchestration and Workflows(#contents.xhtmlch05a)
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Error or failure?
- *Data Engineering Design Patterns* — Bartosz Konieczny — Error Management Design Patterns › Pipeline trigger
- *Data Engineering Design Patterns* — Bartosz Konieczny — Time Detectors › Examples
- *Data Pipelines Pocket Reference* — James Densmore — Data Validation in Pipelines › When to Halt a Pipeline, When to Warn and Continue
- *Data Pipelines with Apache Airflow* — Bas P. Harenslak, Julian de Ruiter — Anatomy of an Airflow DAG
- *Azure Data Factory Cookbook* — Xenia Ireton et al. — Monitoring and Troubleshooting Data Pipelines › Investigating failures -- running in debug mode
- *Azure Data Factory Cookbook* — Xenia Ireton et al. — Monitoring and Troubleshooting Data Pipelines › Rerunning activities
- *Azure Data Factory Cookbook* — Xenia Ireton et al. — Monitoring and Troubleshooting Data Pipelines › How it works...
- *Azure Data Factory Cookbook* — Xenia Ireton et al. — Monitoring and Troubleshooting Data Pipelines › There\'s more...

