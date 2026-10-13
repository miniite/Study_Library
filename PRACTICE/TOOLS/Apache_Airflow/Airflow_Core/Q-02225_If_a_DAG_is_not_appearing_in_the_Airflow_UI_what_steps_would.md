---
type: "practice"
difficulty: "Intermediate"
---

# If a DAG is not appearing in the Airflow UI what steps would you take to troubleshoot?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DAG Design and Dependencies](<../../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose why a DAG isn't showing up in the Airflow UI, you'll need to follow a systematic approach that covers both configuration and operational aspects. This involves checking the Airflow scheduler status, verifying the DAG file's path and content, ensuring proper database synchronization, and reviewing logs for any errors or warnings.

Triaging the Problem

Firstly, clarify what exactly is meant by "not appearing." Is it a case of the DAG being completely missing from the UI? Or perhaps it’s showing up but not running as expected? For this explanation, let's assume you are trying to troubleshoot why a DAG that used to be visible in the Airflow UI has suddenly disappeared.

Step-by-Step Troubleshooting

Check the Airflow Web UI:

Confirm if there is no entry for the DAG at all or if it’s just missing from its usual location.

Verify the DAG File Location and Content:

Navigate to the directory where your DAG files are stored, e.g., /airflow/dags/.
Ensure that the DAG file still exists in this location with the correct filename and content (e.g., my_dag.py).

Review Airflow Scheduler Status:

Check if the Airflow scheduler is running using:
BASH
ps aux | grep airflow-scheduler
If it’s not running, start the scheduler by executing:
BASH
airflow webserver &
airflow scheduler &

Check Airflow Logs:

Look at the logs for both the scheduler and worker processes to see if there are any error messages or warnings that could indicate why the DAG isn’t being picked up.
Common places to check:
BASH
tail -f /path/to/airflow/logs/scheduler.log
tail -f /path/to/airflow/logs/worker.log

Database Synchronization:

Ensure that the metadata database is properly synchronized with your DAG files.
Run the list_dags command from the Airflow CLI to see if the DAG shows up:
BASH
airflow list_dags --json > /tmp/dags.json
Compare this output with what you expect. If it’s missing, there might be an issue with your database or how the metadata is being loaded.

Permissions and Ownership:

Verify that the Airflow user has proper permissions to read the DAG files.
Check ownership and file permissions:
BASH
ls -l /airflow/dags/

Configuration File (airflow.cfg):

Ensure your airflow.cfg is correctly configured, particularly the paths related to DAG directories and databases.

#### Closing Summary
and Common Mistakes
The primary issue often lies in misconfiguration or missing files. However, it could also be due to database synchronization problems.

#### Common mistakes
Forgetting to restart the Airflow webserver or scheduler after making changes.
Not checking the logs for detailed error messages.
Misconfiguring the airflow.cfg file.
Additional Considerations
If the DAG used to work but stopped suddenly, consider recent changes in configurations or dependencies.
Ensure that there are no typos or syntax errors in your DAG files. These can prevent Airflow from recognizing and running them.

