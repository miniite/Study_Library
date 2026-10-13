---
type: "practice"
difficulty: "Deep"
---

# Design a feature store that serves ML features online at low latency and computes them offline consistently — how do you avoid train/serve skew and keep online features fresh from streaming updates?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Real-time Counters](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Real-time_Counters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a feature store that serves ML features online with minimal latency while also computing these features offline, we need to focus on avoiding train/serve skew and ensuring that our online features remain up-to-date from streaming updates. The core idea is to decouple the training data flow from the serving data flow, using techniques like change streams, watermarking, and incremental computation.

#### Triaging the Question

The question is essentially asking how we can design a system where:

ML Models are trained on historical data or real-time updates.
Online Features are served at low latency to power ML applications.
Synchronization Mechanisms exist to ensure that both training and serving processes use consistent feature definitions.
Key Assumptions
The system needs to handle high volumes of streaming data efficiently.
There is a need for consistency between the features used in training and those served online.
Latency for online queries is critical, but not as critical as serving accuracy.
We have an existing pipeline for computing features offline.

#### First Principles Solution
Step 1: Define Feature Definitions

First, define the feature definitions. These should be versioned and documented to ensure consistency between training and serving environments. This can involve using a schema registry or a metadata management system to track changes in feature definitions.

FeatureDefinition
+id
+name
+version
+definition
+creationTime
+lastUpdateTime
Step 2: Offline Computation Pipeline

The offline computation pipeline should compute features incrementally, using change streams to capture updates in real-time. This ensures that the latest data is used for training without introducing significant latency.

PYTHON
# Example pseudocode for an incremental feature computation system
def process_changes(changes):
    for change in changes:
        # Process each change (e.g., update feature vectors)
        pass

def start_computation_pipeline():
    while True:
        changes = read_change_stream()
        process_changes(changes)
Step 3: Online Feature Serving

The online feature serving system should use the most recent version of feature definitions. This can be achieved by timestamping queries or using a watermark mechanism to ensure that only up-to-date features are served.

WatermarkManager
ChangeStream
FeatureStore
UserApplication
WatermarkManager
ChangeStream
FeatureStore
UserApplication
Query(feature_id, as_of_time)
Get latest watermark
latest_watermark
Fetch changes after latest_watermark
Changes
Return features
Step 4: Synchronization and Validation

A synchronization process should validate that the training data matches the serving data. This can be done using periodic audits or by setting up a feedback loop where discrepancies are flagged and addressed.

ValidationService
ServingPipeline
TrainingPipeline
ValidationService
ServingPipeline
TrainingPipeline
Validate features against latest definition
Report any mismatches

#### Closing Summary

By decoupling the offline and online feature computation processes, we can ensure that train/serve skew is minimized. The use of change streams for real-time updates and watermarking ensures that both training and serving environments are using consistent feature definitions. Regular validation helps catch and correct discrepancies early.

#### Common Mistakes/Follow-Ups
Ignoring Change Streams: Failing to leverage change streams can lead to stale data being used in online features.
Lack of Watermarking: Without a watermark mechanism, it's easy for the system to serve old or inconsistent feature versions.
Synchronization Gaps: Not having a robust validation and synchronization process can result in undetected discrepancies between training and serving environments.

**Comparison:**

```
FeatureDefinition	id	name	version	definition
1	1	user_activity	2	Sum of user sessions in the past week
```

**Code:**

```python
# Pseudocode for incremental computation pipeline
def process_changes(changes):
    for change in changes:
        # Process each change (e.g., update feature vectors)
        pass

def start_computation_pipeline():
    while True:
        changes = read_change_stream()
        process_changes(changes)
```

**Example input / output:**

```text
Input:
- Change stream with updates from a user session

Output:
- Updated feature vectors for user_activity
```

**Data:**

```json
{
  "FeatureDefinition": {
    "id": 1,
    "name": "user_activity",
    "version": 2,
    "definition": "Sum of user sessions in the past week"
  }
}
```

**Flow:**

1. WatermarkManager
2. ChangeStream
3. FeatureStore
4. UserApplication
5. WatermarkManager
6. ChangeStream
7. FeatureStore
8. UserApplication
9. Query(feature_id, as_of_time)
10. Get latest watermark
11. latest_watermark
12. Fetch changes after latest_watermark
13. Changes
14. Return features

