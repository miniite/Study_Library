---
type: "practice"
difficulty: "Deep"
---

# How do you orchestrate an SCD Type 2 load in Airflow so a mid-batch failure doesn't leave the dimension half-updated?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To handle a mid-batch failure during an SCD Type 2 load in Airflow, we need to ensure that the dimension records are fully updated, even if the batch process fails partway through. This involves setting up proper error handling and retry mechanisms within our Airflow DAGs, and managing state transitions effectively.

#### Triaging the Question

The question is asking how to design a fault-tolerant SCD Type 2 load in Airflow that guarantees the completion of dimension updates even if a batch fails mid-way. The core issue here is ensuring consistency and integrity of the data while handling potential failures during the update process. We need to consider error handling, state management, and ensuring that all partial states are correctly managed.

#### Assumptions
Airflow DAGs: Assume we have an Airflow environment where SCD Type 2 loads are performed.
Batch Processing: The batch processing involves updating a dimension table based on the current state of the source system.
Error Handling: We need to handle errors gracefully and ensure that partial updates do not occur.
Step-by-Step Solution

Define the DAG Structure:

We will create an Airflow DAG with multiple tasks to update different parts of the dimension table in a sequential manner.
Each task is responsible for updating a specific key or set of keys in the dimension table.

Task Dependencies and Retries:

Use python_callable functions within tasks to execute the SCD Type 2 logic.
Add appropriate retries using retries parameter in Airflow operators, ensuring that if a task fails, it will retry with exponential backoff.
PYTHON
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

def update_dimension_table(key):
    # Logic to update dimension table based on the key
    pass

default_args = {
    'owner': 'data_engineer',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'scd_type_2_load',
    default_args=default_args,
    description='Load dimension table using SCD Type 2 strategy',
    schedule_interval=None,
    catchup=False
)

with dag:
    for key in dimension_keys:
        task_id = f'update_dimension_{key}'
        update_task = PythonOperator(
            task_id=task_id,
            python_callable=update_dimension_table,
            op_kwargs={'key': key},
            provide_context=True,
            retries=3
        )
        if key == dimension_keys[0]:
            update_task.set_downstream(all(tasks))
        else:
            previous_task = tasks[-1]
            update_task.set_upstream(previous_task)

State Management:

Use a database or an external storage to maintain the state of each task and ensure that partial updates are not committed.
PYTHON
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()
engine = create_engine('postgresql://username:password@localhost/dbname')
Session = sessionmaker(bind=engine)

class TaskState(Base):
    __tablename__ = 'task_state'
    id = Column(Integer, primary_key=True)
    task_id = Column(String, unique=True)
    status = Column(String)

def update_task_status(task_id, status):
    session = Session()
    state = session.query(TaskState).filter_by(task_id=task_id).first()
    if not state:
        new_state = TaskState(task_id=task_id, status=status)
        session.add(new_state)
    else:
        state.status = status
        session.commit()

update_task_status(task_id, 'in_progress')
# Update dimension table...
update_task_status(task_id, 'completed')

Error Handling and Logging:

Implement logging to capture any errors or partial updates.
PYTHON
import logging

def update_dimension_table(key):
    try:
        update_dimension_table_logic(key)
        logging.info(f'Successfully updated dimension for key: {key}')
    except Exception as e:
        logging.error(f'Error updating dimension for key: {key}. Error: {str(e)}')
        update_task_status(task_id, 'failed')

#### Closing Summary

By structuring our Airflow DAG to handle tasks sequentially and using retries with proper state management, we can ensure that even if a batch fails mid-way, the dimension table will be fully updated. The key is to maintain consistency through robust error handling and state tracking.

#### Common Mistakes/Follow-Ups
Overlooking State Management: Failing to use an external database or storage for maintaining task states can lead to partial updates.
Insufficient Error Handling: Not properly logging errors or handling retries can result in incomplete data or repeated failed tasks.
Improper DAG Design: Incorrect task dependencies and order can cause unexpected behavior and potential data integrity issues.

**Comparison:**

| Task ID | Status |
|---------|--------|
| update_dimension_1  | completed |
| update_dimension_2  | completed |
| update_dimension_3  | completed |

**Code:**

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()
engine = create_engine('postgresql://username:password@localhost/dbname')
Session = sessionmaker(bind=engine)

class TaskState(Base):
    __tablename__ = 'task_state'
    id = Column(Integer, primary_key=True)
    task_id = Column(String, unique=True)
    status = Column(String)

def update_task_status(task_id, status):
    session = Session()
    state = session.query(TaskState).filter_by(task_id=task_id).first()
    if not state:
        new_state = TaskState(task_id=task_id, status=status)
        session.add(new_state)
    else:
        state.status = status
        session.commit()

def update_dimension_table(key):
    try:
        # Logic to update dimension table based on the key
        pass
    except Exception as e:
        logging.error(f'Error updating dimension for key: {key}. Error: {str(e)}')
        update_task_status(task_id, 'failed')

default_args = {
    'owner': 'data_engineer',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'scd_type_2_load',
    default_args=default_args,
    description='Load dimension table using SCD Type 2 strategy',
    schedule_interval=None,
    catchup=False
)

def update_dimension_table_logic(key):
    # Implementation of the logic to update dimension table based on key

dimension_keys = [1, 2, 3]  # Example keys

with dag:
    for key in dimension_keys:
        task_id = f'update_dimension_{key}'
        update_task = PythonOperator(
            task_id=task_id,
            python_callable=update_dimension_table_logic,
            op_kwargs={'key': key},
            provide_context=True,
            retries=3
        )
        if key == dimension_keys[0]:
            update_task.set_downstream(all(tasks))
        else:
            previous_task = tasks[-1]
            update_task.set_upstream(previous_task)
```

**Example input / output:**

```text
Input (Partial failure during batch processing):
- Task 'update_dimension_1' completes successfully.
- Task 'update_dimension_2' fails due to an unexpected error.

Output (Post-retry and state management):
- Task 'update_dimension_2' retries and successfully completes after the issue is resolved.
- All tasks are marked as completed in the database, ensuring full data integrity.
```

**Data:**

```yaml
tasks:
  - task_id: 'update_dimension_1'
    status: in_progress
  - task_id: 'update_dimension_2'
    status: failed
  - task_id: 'update_dimension_3'
    status: completed
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Task 1
7. Update Dimension Table
8. Error?
9. Log Error and Retry Task
10. Task 2
11. Update Dimension Table
12. Error?
13. Log Error and Retry Task
14. End

