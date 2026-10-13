---
type: "practice"
difficulty: "Intermediate"
---

# How would you schedule and run periodic jobs reliably in a Python service (APScheduler vs Celery beat vs external cron), and how do you avoid duplicate firing under multiple replicas?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dead Letter and Replay Systems](<../../../../TOOLS/Python/Python_Language/Dead_Letter_and_Replay_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To schedule and run periodic jobs reliably in a Python service, we need to consider various factors such as job reliability, ease of use, scalability, and fault tolerance. In this scenario, I would discuss three common approaches: using APScheduler within the application itself, utilizing Celery Beat, or leveraging an external cron-based solution like systemd timers.

The goal is to ensure that periodic jobs are executed reliably even when there are multiple replicas running. This typically involves understanding how each approach handles job scheduling and deduplication under such conditions.

**Code:**

```python
# Example of APScheduler setup in Python
from apscheduler.schedulers.background import BackgroundScheduler

def periodic_task():
    print("Task executed")

scheduler = BackgroundScheduler()
scheduler.add_job(periodic_task, 'interval', minutes=1)
scheduler.start()

# Example Celery Beat configuration snippet
CELERY_BEAT_SCHEDULE = {
    'periodic-task': {
        'task': 'myapp.tasks.periodic_task',
        'schedule': 60.0,  # every minute
    },
}
```

**Flow:**

1. Job registered and started
2. Yes
3. No
4. Simplified setup but limitedscalability
5. Yes
6. No
7. APScheduler
8. Multiple replicas?
9. No deduplication mechanism,potential race condition
10. Celery Beat with distributedlock, reliable execution acrossreplicas
11. Cron-based timers likesystemd
12. Requires external system
13. Systemd timers or similar,good for simple cases
14. Manual configuration andmanagement required

