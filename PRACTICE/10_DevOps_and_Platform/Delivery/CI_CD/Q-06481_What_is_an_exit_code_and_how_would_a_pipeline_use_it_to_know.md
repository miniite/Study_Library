---
type: "practice"
difficulty: "Basic"
---

# What is an exit code and how would a pipeline use it to know whether the alert-rule validation step passed?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline as Code](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pipeline_as_Code.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Exit codes are used in CI/CD pipelines to indicate the success or failure of commands or scripts executed during various stages, such as alert-rule validation. A pipeline uses these exit codes to determine if a particular step passed or failed and to take appropriate actions based on that result.

To understand this fully, let's break down what an exit code is and how it can be utilized in the context of CI/CD pipelines.

**Code:**

```python
# Example Python script for alert rule validation
import subprocess

def validate_alert_rule(file_path):
    # Execute a command (for example, running a Python script)
    process = subprocess.run(["python", file_path], capture_output=True, text=True)

    # The return code of the executed command is stored in `process.returncode`
    if process.returncode == 0:
        print("Alert rule validation passed.")
        return True
    else:
        print(f"Alert rule validation failed with exit code {process.returncode}.")
        return False

# Usage of validate_alert_rule function
validation_result = validate_alert_rule('path/to/your/alert/rule/validation/script.py')
if not validation_result:
    # If the alert rule validation fails, further actions can be taken here (e.g., sending a failure notification)
    pass
```

**Example input / output:**

```text
# Example command and its potential exit codes
$ python validate_alert_rule.py

# If the script runs successfully, it might output:
Alert rule validation passed.

# If there's an error in the script or its dependencies are missing, it might output:
Alert rule validation failed with exit code 1.
```

**Flow:**

1. Yes
2. No
3. Pipeline Start
4. Execute Alert Rule ValidationScript
5. Exit Code == 0?
6. Pipeline Proceeds to Next Step
7. Send Failure Notification andPipeline Stalls
8. Audit Log Entry

