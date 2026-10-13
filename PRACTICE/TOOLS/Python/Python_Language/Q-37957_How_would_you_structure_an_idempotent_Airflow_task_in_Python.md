---
type: "practice"
difficulty: "Deep"
---

# How would you structure an idempotent Airflow task in Python so that re-running a failed DAG run or a backfill doesn't double-write to the target table?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Airflow Tasks](<../../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To structure an idempotent Airflow task in Python, we need to ensure that rerunning a failed DAG (Data Acquisition and Processing) run or a backfill does not lead to double-writing to the target database table. This involves detecting whether data has already been processed for a given period or record, and only processing new or changed records.

#### Triage

The question is probing how to implement idempotency in Airflow tasks using Python. Key points include:

Detecting whether data has already been processed.
Allowing backfills without causing duplicates.
Ensuring the task does not double-write to the target table.

#### Assumptions
We have a database with a primary key or unique identifier for each record.
The DAG can be run periodically (e.g., daily, hourly) and also as a backfill job.
We need to track which records have already been processed in the recent past.

#### First-Principles Solution
Identify Unique Identifiers: Use unique identifiers from the source data or create them if necessary.
Track Processed Records: Store a record of processed keys in a database table.
Check Before Processing: Before inserting/processing, check if the key is already present in the tracked records.
Handle Backfills: Implement logic to handle backfilling without processing already tracked records.
Detailed Steps
Database Design for Tracking:
Create a simple tracking table (e.g., processed_records) with columns: id, record_key, and processing_time.
Check Function in Python Task:
PYTHON
def is_processed(record_key, session):
    return session.query(ProcessedRecords).filter_by(record_key=record_key).first() is not None

@task.python(task_id="process_data", provide_context=True)
def process_data(context, source_records):
    session = context['session']

    for record in source_records:
        if not is_processed(record.key, session):
            # Process the record
            process_record(record)

            # Track processed record
            tracked_record = ProcessedRecords(record_key=record.key, processing_time=datetime.utcnow())
            session.add(tracked_record)
            session.commit()
Backfill Handling:
When backfilling, ensure that only records within the specified date range are processed.
Use a query to filter out already tracked records before inserting them into the task.
Full Solution
PYTHON
from sqlalchemy import create_engine, Column, Integer, String, DateTime
from sqlalchemy.orm import sessionmaker
from airflow.models import Base

class ProcessedRecords(Base):
    __tablename__ = 'processed_records'
    id = Column(Integer, primary_key=True)
    record_key = Column(String, unique=True)
    processing_time = Column(DateTime)

def is_processed(record_key, session):
    return session.query(ProcessedRecords).filter_by(record_key=record_key).first() is not None

@task.python(task_id="process_data", provide_context=True)
def process_data(context, source_records):
    engine = create_engine('sqlite:///example.db')
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()

    for record in source_records:
        if not is_processed(record.key, session):
            # Process the record
            print(f"Processing {record.key}")

            # Track processed record
            tracked_record = ProcessedRecords(record_key=record.key, processing_time=datetime.utcnow())
            session.add(tracked_record)
            session.commit()

#### Closing Summary

By structuring your Airflow task to include a check for previously processed records and tracking these in a database table, you ensure that the task is idempotent. This approach handles both regular runs and backfills correctly.

#### Common Mistakes/Follow-Ups
Failing to Initialize Tracking Table: Not setting up the ProcessedRecords table can lead to errors.
Ignoring Backfill Logic: When performing a backfill, forgetting to filter out already processed records can cause double-processing.
Incorrect Data Key: Using an incorrect unique key for tracking can result in false negatives or positives.

**Code:**

```python
from sqlalchemy import create_engine, Column, Integer, String, DateTime
from sqlalchemy.orm import sessionmaker
from airflow.models import Base

class ProcessedRecords(Base):
    __tablename__ = 'processed_records'
    id = Column(Integer, primary_key=True)
    record_key = Column(String, unique=True)
    processing_time = Column(DateTime)

def is_processed(record_key, session):
    return session.query(ProcessedRecords).filter_by(record_key=record_key).first() is not None

@task.python(task_id="process_data", provide_context=True)
def process_data(context, source_records):
    engine = create_engine('sqlite:///example.db')
    Base.metadata.create_all(engine)
    Session = sessionmaker(bind=engine)
    session = Session()

    for record in source_records:
        if not is_processed(record.key, session):
            print(f"Processing {record.key}")
            tracked_record = ProcessedRecords(record_key=record.key, processing_time=datetime.utcnow())
            session.add(tracked_record)
            session.commit()
```

