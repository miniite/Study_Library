---
type: "practice"
difficulty: "Deep"
---

# How do you match delayed ground-truth labels back to earlier FastAPI predictions to compute rolling live accuracy, and where does the join happen so it does not slow the serving path?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Monitoring Model Quality in APIs](<../../../../TOOLS/Python/FastAPI/Monitoring_Model_Quality_in_APIs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to understand how to efficiently match delayed ground-truth labels back to earlier FastAPI predictions in order to compute rolling live accuracy without significantly impacting the serving path. The key idea is to implement a robust, scalable system that minimizes latency while ensuring accurate and timely updates.

In summary, our approach will involve:

Storing prediction results and corresponding timestamps.
Designing an efficient join mechanism between predictions and ground-truth labels.
Implementing a background process for accuracy computation.
Ensuring the serving path remains unaffected by these operations.

Let's break down each step in detail:

#### Triage
What is Being Probed

The interviewer is probing your understanding of:

FastAPI architecture and its limitations regarding performance.
Data storage and retrieval strategies that minimize latency.
Background processing techniques for accuracy computation.
Scalability considerations to ensure the system can handle high traffic without degradation.

#### Assumptions
The ground-truth labels are delayed by a few seconds to minutes after predictions.
The volume of data is large, necessitating efficient storage and retrieval mechanisms.
We need to maintain real-time accuracy metrics without impacting prediction latency.
Step-by-Step Solution
1. Storing Prediction Results

We store each FastAPI prediction result along with a timestamp in a database that supports fast read/write operations. For this example, we'll use PostgreSQL due to its robustness and wide adoption.

SQL
CREATE TABLE predictions (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    model_version INT NOT NULL,
    prediction_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    predicted_label VARCHAR(255)
);
2. Efficient Join Mechanism

To match delayed ground-truth labels back to predictions, we use a database view or a materialized view (if the delay is consistent). This allows us to perform joins efficiently without blocking the serving path.

SQL
CREATE MATERIALIZED VIEW prediction_ground_truth AS
SELECT p.id, g.label
FROM predictions p
JOIN ground_truth_labels g ON p.user_id = g.user_id AND p.prediction_time BETWEEN g.start_time AND g.end_time;

Alternatively, we can use a database view that refreshes periodically:

SQL
CREATE OR REPLACE VIEW prediction_ground_truth AS
SELECT p.id, g.label
FROM predictions p
JOIN ground_truth_labels g ON p.user_id = g.user_id AND p.prediction_time BETWEEN g.start_time AND g.end_time;
3. Background Process for Accuracy Computation

We implement a background process that runs periodically to compute and update rolling live accuracy metrics. This process can be triggered using a cron job or an event-driven architecture.

PYTHON
import datetime

def compute_accuracy():
    # Fetch predictions from the database within the last hour
    predictions = db.fetch_predictions(datetime.datetime.now() - datetime.timedelta(hours=1))

    correct_count = 0
    total_count = len(predictions)

    for pred in predictions:
        if pred['predicted_label'] == pred['ground_truth_label']:
            correct_count += 1

    accuracy = correct_count / total_count
    db.update_accuracy(accuracy)

# Example cron job setup
import schedule, time

def run_job():
    compute_accuracy()
    schedule.every(60).minutes.do(run_job)  # Run every hour

run_job()
4. Ensuring Serving Path Remains Unaffected

To ensure the serving path remains unaffected, we can use a separate database or a different schema for storing ground-truth labels and predictions. This separation helps in managing resources efficiently.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, by leveraging efficient data storage, background processing, and proper resource management, we can maintain real-time accuracy metrics without impacting the serving path significantly. Key mistakes to avoid include:

Using a single table for all operations, leading to performance bottlenecks.
Failing to consider periodic updates for materialized views or ensuring they are refreshed regularly.
Not separating concerns between prediction and accuracy computation, which could lead to resource contention.

#### Follow-up
questions might involve discussing the trade-offs between real-time and historical accuracy metrics, exploring alternative storage solutions like Apache Kafka for streaming data, or discussing how to scale this system for very high traffic scenarios.

**Comparison:**

| User ID | Start Time | End Time | Label | |---------|----------------------|--------------------| | 101 | 2023-09-15 14:28:00 | 2023-09-15 14:32:00 | A | | 102 | 2023-09-15 14:30:00 | 2023-09-15 14:34:00 | B |

**Code:**

```python
import datetime

def compute_accuracy():
    # Fetch predictions from the database within the last hour
    predictions = db.fetch_predictions(datetime.datetime.now() - datetime.timedelta(hours=1))

    correct_count = 0
    total_count = len(predictions)

    for pred in predictions:
        if pred['predicted_label'] == pred['ground_truth_label']:
            correct_count += 1

    accuracy = correct_count / total_count
    db.update_accuracy(accuracy)

# Example cron job setup
import schedule, time

def run_job():
    compute_accuracy()
    schedule.every(60).minutes.do(run_job)  # Run every hour

run_job()
```

**Example input / output:**

```text
# Example input for accuracy computation
predictions = [
    {'id': 1, 'user_id': 101, 'model_version': 2, 'predicted_label': 'A', 'prediction_time': datetime.datetime(2023, 9, 15, 14, 30)},
    {'id': 2, 'user_id': 102, 'model_version': 2, 'predicted_label': 'B', 'prediction_time': datetime.datetime(2023, 9, 15, 14, 31)}
]

# Example ground-truth labels
ground_truth_labels = [
    {'user_id': 101, 'start_time': datetime.datetime(2023, 9, 15, 14, 28), 'end_time': datetime.datetime(2023, 9, 15, 14, 32), 'label': 'A'},
    {'user_id': 102, 'start_time': datetime.datetime(2023, 9, 15, 14, 30), 'end_time': datetime.datetime(2023, 9, 15, 14, 34), 'label': 'B'}
]

# Computed accuracy
accuracy = compute_accuracy(predictions, ground_truth_labels)
print(f"Accuracy: {accuracy}")
```

**Data:**

```sql
CREATE TABLE predictions (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    model_version INT NOT NULL,
    prediction_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    predicted_label VARCHAR(255)
);

CREATE MATERIALIZED VIEW prediction_ground_truth AS
SELECT p.id, g.label
FROM predictions p
JOIN ground_truth_labels g ON p.user_id = g.user_id AND p.prediction_time BETWEEN g.start_time AND g.end_time;
```

**Flow:**

1. No
2. Yes
3. Fetch Predictions
4. Are Labels Delayed?
5. Merge with Ground Truth
6. Use Materialized View
7. Compute Accuracy
8. Update Accuracy Metrics

