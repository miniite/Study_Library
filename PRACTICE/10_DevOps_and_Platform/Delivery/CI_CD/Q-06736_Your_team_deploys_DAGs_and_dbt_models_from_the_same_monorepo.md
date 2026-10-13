---
type: "practice"
difficulty: "Deep"
---

# Your team deploys DAGs and dbt models from the same monorepo; how would you build affected-target detection so a change to one model tests only its lineage, not the whole warehouse?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Infrastructure Drift Detection](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Infrastructure_Drift_Detection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To address this challenge, we'll implement an incremental dependency analysis pipeline using a combination of static and dynamic analysis techniques. This approach ensures that changes to one model trigger only relevant downstream tests, significantly reducing test execution time and improving overall efficiency.

#### Triage

The question revolves around building affected-target detection for CI/CD pipelines where both DAGs (Apache Airflow) and dbt models are stored in a monorepo. The goal is to minimize redundant testing by accurately identifying the lineage of changes, thereby executing only necessary tests.

#### Assumptions
Monorepo Structure: Both DAGs and dbt models coexist within the same Git repository.
Build System: Continuous Integration (CI) system like Jenkins or GitHub Actions handles the build and deployment process.
Dependency Mapping: The project maintains a clear mapping of dependencies between DAGs, dbt models, and database tables.

#### First-Principles Solution
Step 1: Dependency Graph Construction
Static Analysis:
Parse dbt model files to extract dependency information.
Use Airflow metadata to map task dependencies in DAGs.
Dynamic Analysis:
Integrate a system that can dynamically analyze changes during the build process.
Step 2: Change Detection
Git Hooks: Implement Git hooks to detect changes and trigger static analysis immediately upon commit.
Build Scripts: Enhance existing CI scripts to capture which specific files were modified.
Step 3: Lineage Tracing
Dependency Graph Traversal:
For dbt, traverse the dependency graph starting from the changed model, identify direct and indirect dependencies.
For Airflow tasks, trace task execution paths based on DAG definitions.
Step 4: Test Selection
Test Identification:
Use the lineage information to select tests that are relevant to the modified models or tasks.
Execution Control:
Ensure that only selected tests run in the CI pipeline, optimizing resource utilization and test time.
Detailed Implementation
Static Analysis for dbt Models
dbt Dependency Extraction: Utilize dbt-docs or custom scripts to parse metadata from .sql files.
Dependency Graph Construction:
PYTHON
import networkx as nx

def build_dbt_dependency_graph(models):
    G = nx.DiGraph()
    for model in models:
        # Extract dependencies and add edges
        pass
    return G
Dynamic Analysis During CI Build
Git Change Detection:
BASH
git diff --name-only HEAD~1 | grep '\.sql$'
Dependency Graph Update:
Re-run static analysis if significant changes are detected.
Otherwise, use cached graph for efficiency.
Lineage Tracing in Airflow DAGs
DAG Metadata Parsing:
PYTHON
from airflow.models import DagModel

def parse_dag_dependencies(dag_id):
    dag = session.query(DagModel).filter_by(dag_id=dag_id).first()
    if not dag:
        return []
    task_ids = [task.task_id for task in dag.tasks]
    # Parse task dependencies
    pass
Test Selection and Execution
Test Suite Configuration:
Configure dbt tests to run based on lineage.
Use Airflow’s TriggerDagRunOperator to selectively execute downstream tasks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By implementing this solution, we can significantly reduce the overhead of testing in a monorepo environment. Key challenges include maintaining accurate dependency mappings and optimizing test execution efficiency. Potential follow-ups could involve integrating more sophisticated machine learning techniques for dynamic analysis or exploring distributed systems to parallelize the build process.

#### Common mistakes
include overcomplicating the initial design without properly validating assumptions, neglecting edge cases (e.g., changes in shared dependencies), and failing to scale the system as the repository grows.

**Comparison:**

```
Model Name	Dependencies
user_activity	another_table
send_email_alerts	process_user_data
```

**Code:**

```python
import networkx as nx

def build_dbt_dependency_graph(models):
    G = nx.DiGraph()
    for model in models:
        # Extract dependencies and add edges
        pass
    return G

def parse_dag_dependencies(dag_id):
    dag = session.query(DagModel).filter_by(dag_id=dag_id).first()
    if not dag:
        return []
    task_ids = [task.task_id for task in dag.tasks]
    # Parse task dependencies
    pass
```

**Example input / output:**

```text
# Git change detection
git diff --name-only HEAD~1 | grep '\.sql$'
# Output: user_activity.sql

# Dependency graph update
dbt dependency update
```

**Data:**

```json
{
  "dbt_model": {
    "name": "user_activity",
    "depends_on": ["another_table"]
  },
  "dag_task": {
    "task_id": "process_user_data",
    "downstream_tasks": ["send_email_alerts"]
  }
}
```

**Flow:**

1. User Commits Change
2. Git Hook Triggers
3. Dependency Extraction
4. Static Analysis Update
5. Traverse Dependency Graph
6. Test Selection & ExecutionControl
7. DAG Task Modification
8. Git Hook Triggers
9. Task Dependency Parsing
10. Dynamic Analysis Update
11. Traverse DAG Execution Paths
12. Trigger Downstream Tasks

