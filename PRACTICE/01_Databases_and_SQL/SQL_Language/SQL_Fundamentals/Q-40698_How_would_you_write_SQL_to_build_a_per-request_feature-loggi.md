---
type: "practice"
difficulty: "Deep"
---

# How would you write SQL to build a per-request feature-logging table so you can later reconstruct exactly what the model saw at inference time for debugging and replay?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of logging features seen by an ML model during inference, we need to design a SQL schema that captures all relevant information about each request and the corresponding feature values. This approach ensures that we can reconstruct the exact input provided to the model for debugging and replay purposes.

#### Triage

The core task here is to create a detailed log table in SQL that records every feature value for each inference request. The key aspects to consider include:

Request Identification: Unique identifier for each request.
Feature Values: Individual features passed during the request.
Timestamps: When the request was made and when processing started/ended.
Model Version: Which version of the model was used.

#### First Principles Solution

We will design a schema with tables that capture requests, feature values, and any additional metadata. The requests table will track the overall request process, while the features table will log individual feature details for each request.

Schema Design
SQL
CREATE TABLE IF NOT EXISTS requests (
    id SERIAL PRIMARY KEY,
    request_id VARCHAR(50) UNIQUE NOT NULL,  -- Unique identifier for the request
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- Timestamp when the request started
    end_time TIMESTAMP,  -- Optional: timestamp when the request ended (if available)
    model_version VARCHAR(100),  -- Version of the model used
    status VARCHAR(20) CHECK(status IN ('completed', 'failed')),  -- Status of the request
    FOREIGN KEY (request_id) REFERENCES features(request_id) ON DELETE CASCADE  -- Ensures referential integrity
);

CREATE TABLE IF NOT EXISTS features (
    id SERIAL PRIMARY KEY,
    request_id VARCHAR(50),  -- Foreign key to requests table
    feature_name VARCHAR(100) NOT NULL,  -- Name of the feature
    feature_value TEXT NOT NULL,  -- Value of the feature (text or JSON format)
    inserted_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- Timestamp when this record was inserted
    FOREIGN KEY (request_id) REFERENCES requests(request_id) ON DELETE CASCADE  -- Ensures referential integrity
);

#### Implementation
and Dry-Run

Assume we have a prediction request with the following features:

feature1: "value1"
feature2: "value2"

The SQL insert statements would look like this:

SQL
-- Inserting a new request
INSERT INTO requests (request_id, model_version) VALUES ('req-001', 'v1.0');

-- Logging features for the same request
INSERT INTO features (request_id, feature_name, feature_value)
VALUES ('req-001', 'feature1', 'value1'),
       ('req-001', 'feature2', 'value2');

#### Closing Summary
and Common Mistakes

To fully reconstruct what the model saw during inference, we need to ensure that:

Referential Integrity: The requests table ensures that each feature is associated with a valid request.
Timestamps: Useful for understanding the timeline of events.
Model Version: Important for debugging specific versions.

#### Common mistakes
include:

Failing to enforce referential integrity, leading to inconsistent data.
Missing timestamps or having them in an incorrect format.
Not logging model version details.
Common Follow-Ups
How would you handle scenarios where multiple models are used simultaneously?
What if a feature value is missing or null?
How do you ensure this table scales with high request volumes?

**Comparison:**

```
has
REQUESTS
int
id
varchar(50)
request_id
timestamp
start_time
timestamp
end_time
varchar(100)
model_version
varchar(20)
status
FEATURES
int
id
varchar(50)
request_id
varchar(100)
feature_name
text
feature_value
timestamp
inserted_time
```

**Code:**

```sql
CREATE TABLE IF NOT EXISTS requests (
    id SERIAL PRIMARY KEY,
    request_id VARCHAR(50) UNIQUE NOT NULL,  -- Unique identifier for the request
    start_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- Timestamp when the request started
    end_time TIMESTAMP,  -- Optional: timestamp when the request ended (if available)
    model_version VARCHAR(100),  -- Version of the model used
    status VARCHAR(20) CHECK(status IN ('completed', 'failed')),  -- Status of the request
    FOREIGN KEY (request_id) REFERENCES features(request_id) ON DELETE CASCADE  -- Ensures referential integrity
);

CREATE TABLE IF NOT EXISTS features (
    id SERIAL PRIMARY KEY,
    request_id VARCHAR(50),  -- Foreign key to requests table
    feature_name VARCHAR(100) NOT NULL,  -- Name of the feature
    feature_value TEXT NOT NULL,  -- Value of the feature (text or JSON format)
    inserted_time TIMESTAMP DEFAULT CURRENT_TIMESTAMP,  -- Timestamp when this record was inserted
    FOREIGN KEY (request_id) REFERENCES requests(request_id) ON DELETE CASCADE  -- Ensures referential integrity
);
```

**Example input / output:**

```text
-- Inserting a new request
INSERT INTO requests (request_id, model_version) VALUES ('req-001', 'v1.0');

-- Logging features for the same request
INSERT INTO features (request_id, feature_name, feature_value)
VALUES ('req-001', 'feature1', 'value1'),
       ('req-001', 'feature2', 'value2');
```

**Data:**

```json
{
  "requests": [
    {
      "id": 1,
      "request_id": "req-001",
      "start_time": "2023-10-05 14:00:00",
      "end_time": null,
      "model_version": "v1.0",
      "status": "completed"
    }
  ],
  "features": [
    {
      "id": 1,
      "request_id": "req-001",
      "feature_name": "feature1",
      "feature_value": "value1",
      "inserted_time": "2023-10-05 14:00:00"
    },
    {
      "id": 2,
      "request_id": "req-001",
      "feature_name": "feature2",
      "feature_value": "value2",
      "inserted_time": "2023-10-05 14:00:00"
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. User makes a predictionrequest
4. Is the model version new?
5. Insert new entry in `requests`table
6. Log features for the request in`features` table
7. Update existing entry or addnew version of the `requests`table if needed
8. Log features for the updatedrequest in `features` table

