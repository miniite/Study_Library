---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd keep a search index in sync with a primary DB via CDC and detect drift in production. What reconciliation job would you run and how do you avoid it becoming a load spike itself?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a search index is kept up-to-date with a primary database using Change Data Capture (CDC), while also detecting any drift, we need to implement a robust reconciliation mechanism. The key steps involve setting up CDC to capture changes in the primary database and then processing these changes to update the search index. Additionally, we must periodically reconcile the search index against the primary database to detect any discrepancies or drift.

Firstly, let's break down what we are aiming for:

Primary Database: The central source of truth.
Search Index: A secondary system that needs to be kept in sync with the primary database for quick querying and search operations.
CDC: A mechanism that captures data changes as they happen in the primary database.
Reconciliation Job: A process to periodically verify the integrity of the search index.

#### Triaging the Question

The question is probing your understanding of:

How CDC works and how to set it up.
The design and implementation of a reconciliation job.
Strategies to avoid load spikes caused by the reconciliation job.
Error handling and drift detection mechanisms.

#### First-Principles Solution
Step 1: Setting Up CDC

We start by setting up CDC in the primary database. This can be done using various methods depending on the database system:

MySQL: Using binlog to capture changes.
PostgreSQL: Utilizing logical decoding.
MongoDB: Through change streams.

For this example, let's assume a PostgreSQL environment with logical decoding.

Step 2: Implementing CDC in Code

We create a CDC listener that captures changes and sends them to our application. The code snippet below shows how you might set up such a system:

PYTHON
import psycopg2

def setup_cdc(connection_string):
    conn = psycopg2.connect(connection_string)
    cur = conn.cursor()

    # Set up logical decoding
    cur.execute("""
        CREATE EXTENSION IF NOT EXISTS pglogical;
        SELECT pglogical.replication_set_create('rs1');
        SELECT pglogical.stream_create('s1', 'pglogical.logical_slot_0001', 'pglogical.rs1');
    """)
    conn.commit()

    return conn, cur

def start_cdc_listener(conn):
    # Start a thread to listen for changes
    def cdc_listener(cur):
        while True:
            cur.execute("SELECT pglogical.stream_apply_all()")
            time.sleep(5)

    listener_thread = threading.Thread(target=cdc_listener, args=(cur,))
    listener_thread.daemon = True
    listener_thread.start()
Step 3: Processing CDC Changes

The captured changes are then processed to update the search index. This can be done using a batch processing system like Apache Kafka or Amazon Kinesis.

PYTHON
def process_cdc_changes(changelog, indexer):
    for change in changelog:
        if change['action'] == 'INSERT':
            indexer.add(change['data'])
        elif change['action'] == 'UPDATE':
            indexer.update(change['data'])
        elif change['action'] == 'DELETE':
            indexer.remove(change['data'])

# Example usage
indexer = SearchIndex()
process_cdc_changes(changelog, indexer)
Step 4: Running the Reconciliation Job

We need a periodic reconciliation job to ensure that the search index and primary database are in sync. This can be done using a scheduled task like cron jobs or managed services like AWS Lambda.

PYTHON
def reconcile_index(conn):
    with conn.cursor() as cur:
        # Query the primary database for all records
        cur.execute("SELECT * FROM main_table")
        db_records = cur.fetchall()

        # Compare against search index
        indexer_records = indexer.get_all_documents()
        differences = find_differences(db_records, indexer_records)

        if differences:
            log.error(f"Drift detected: {differences}")
            send_alert(differences)

def schedule_reconciliation():
    scheduler.add_job(reconcile_index, 'interval', hours=12, args=[conn])
Avoiding Load Spikes

To avoid the reconciliation job from becoming a load spike:

Batch Processing: Process changes in batches instead of one at a time.
Rate Limiting: Use rate limiting to control how many changes are processed per second.
Asynchronous Execution: Run the reconciliation job asynchronously, possibly using a queue or message broker.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, keeping a search index in sync with a primary database via CDC involves setting up logical decoding, processing changes, and periodically reconciling. To avoid load spikes during reconciliation, use batching, rate limiting, and asynchronous execution.

#### Common mistakes
to avoid:

Not properly handling edge cases such as partial transactions or inconsistent states.
Failing to implement efficient batch processing and rate limiting.
Not setting up appropriate monitoring and alerting for drift detection.

For follow-ups, the interviewer might ask:

How would you handle conflicts between concurrent updates?
What other types of changes (e.g., schema changes) should be considered?

**Code:**

```python
import psycopg2
import threading

def setup_cdc(connection_string):
    conn = psycopg2.connect(connection_string)
    cur = conn.cursor()

    # Set up logical decoding
    cur.execute("""
        CREATE EXTENSION IF NOT EXISTS pglogical;
        SELECT pglogical.replication_set_create('rs1');
        SELECT pglogical.stream_create('s1', 'pglogical.logical_slot_0001', 'pglogical.rs1');
    """)
    conn.commit()

    return conn, cur

def start_cdc_listener(conn):
    def cdc_listener(cur):
        while True:
            cur.execute("SELECT pglogical.stream_apply_all()")
            time.sleep(5)

    listener_thread = threading.Thread(target=cdc_listener, args=(cur,))
    listener_thread.daemon = True
    listener_thread.start()
```

**Data:**

```json
{
  "connection_string": "postgresql://user:password@localhost/database",
  "replication_set": "rs1",
  "stream_name": "s1"
}
```

**Flow:**

1. Setup CDC
2. Start Listener
3. Process Changes
4. Reconciliation Job (Periodic)
5. Avoid Load Spikes

