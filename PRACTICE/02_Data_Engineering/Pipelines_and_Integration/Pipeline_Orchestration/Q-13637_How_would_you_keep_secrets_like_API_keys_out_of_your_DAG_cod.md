---
type: "practice"
difficulty: "Intermediate"
---

# How would you keep secrets like API keys out of your DAG code and git repository?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To keep secrets such as API keys out of our Data Pipeline code and Git repository, we need to adopt a secure and flexible approach. This involves using environment variables or secret management tools, along with proper infrastructure setup. The core idea is to separate the secrets from the pipeline configuration so that they are not accidentally committed to version control.

The question here primarily revolves around ensuring security and best practices for managing sensitive information in your workflow. Here’s a step-by-step solution:

#### Triaging the Question
Identify the Problem: Secrets like API keys should not be hard-coded into your pipeline code or stored in the Git repository.
What is Being Probed? The interviewer is looking for an understanding of secure coding practices and how to implement them effectively.

#### Assumptions
We are using Airflow as our data orchestration tool, although this solution can apply to other tools like Apache Spark as well.
Our pipeline code is written in Python.
Step-by-Step Solution

Environment Variables:

Use environment variables to store secrets. This method ensures that the secrets are not hard-coded and are only available at runtime.

Airflow Configuration Files:

In Airflow, you can use the airflow.cfg file or secrets.yaml files in your DAG directory to reference these environment variables.
For example:
INI
[core]
dag_home = /path/to/dags
conf_dir = /path/to/conf

[secrets]
api_key = $API_KEY

Secret Management Tools:

Use a secret management tool like HashiCorp Vault, AWS Secrets Manager, or Google Cloud Secret Manager to store and retrieve secrets securely.
In your DAG code, you can use these tools to fetch the secrets at runtime.

Infrastructure Setup:

Ensure that sensitive information is stored in a secure environment and accessed only by authorized personnel.
Use role-based access control (RBAC) to manage permissions for accessing secrets.

#### Example
Implementation with Airflow:

PYTHON
from airflow.providers.http.sensors.http import HttpSensor

API_KEY = os.getenv('API_KEY')

def fetch_data():
    response = requests.get('https://api.example.com/data', headers={'Authorization': f'Bearer {API_KEY}'})
    return response.json()

with DAG(dag_id='secrets_example', schedule_interval="@daily", start_date=datetime(2023, 1, 1), catchup=False) as dag:
    fetch_data_task = PythonOperator(
        task_id='fetch_data',
        python_callable=fetch_data
    )

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using environment variables and secret management tools, we can keep secrets out of our DAG code and Git repository. This approach ensures security while maintaining flexibility in how the pipeline is configured.

#### Common Mistakes
Hard-coding secrets directly into the code or configuration files.
Storing sensitive information in plain text within version control systems.
Not using a proper secret management tool, leading to insecure practices.

#### Follow-Ups
How do you handle situations where multiple environments (staging, production) require different API keys?
What are the potential trade-offs between manually setting up environment variables versus leveraging a dedicated secret management service?
Can you discuss how these techniques can be extended to other tools or pipelines beyond Airflow?

**Code:**

```python
from airflow.providers.http.sensors.http import HttpSensor

API_KEY = os.getenv('API_KEY')

def fetch_data():
    response = requests.get('https://api.example.com/data', headers={'Authorization': f'Bearer {API_KEY}'})
    return response.json()

with DAG(dag_id='secrets_example', schedule_interval="@daily", start_date=datetime(2023, 1, 1), catchup=False) as dag:
    fetch_data_task = PythonOperator(
        task_id='fetch_data',
        python_callable=fetch_data
    )
```

