---
type: "practice"
difficulty: "Expert"
---

# Your checkpoint directory on the streaming job grew corrupt after an unclean shutdown; how do you recover without double-counting alerts or losing state?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Recovery and Deduplication](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Streaming_Recovery_and_Deduplication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to recover a Kafka-Spark stream processing job that has encountered corruption in its checkpoint directory due to an unclean shutdown. The goal is to restore the system to a known good state without causing duplicate alerts or data loss. This involves understanding the state management of Spark Structured Streaming and leveraging recovery mechanisms.

Firstly, let's triage the problem: when a Spark job encounters issues like this, it can lead to inconsistencies in the processing pipeline. The checkpoint directory serves as a critical point for recovering from failures by maintaining metadata about which data has been processed. Without proper recovery, we risk either missing some data or processing already handled data again, leading to duplicate alerts.

#### Assumptions

We are using Apache Spark with Structured Streaming.
The job is configured to use checkpointing to persist state periodically.
There is a way to inspect the corrupted checkpoint directory and identify the last successful checkpoint before the unclean shutdown.

Step-by-step solution:

Identify the Last Successful Checkpoint: Inspect the checkpoint directory to find the most recent metadata file that was written before the unclean shutdown. This metadata will contain information about which batches have been processed up until this point.

Stop the Streaming Job Safely: Before making any changes, ensure the current streaming job is stopped safely and no new data is being ingested by accident. You can do this using:

PYTHON
spark.streams.active.stop()

Replay from Last Known Good State: Once the job is stopped, replay the data starting from the last known good state to bring the system back online without double-counting any data.

Update Checkpoint Directory: After replaying, update the checkpoint directory to reflect the new processing boundary and ensure that it does not overwrite existing metadata files, causing inconsistencies.

Monitor for Duplicates: While restoring from a known good state, implement monitoring or logging to detect any duplicate processing early on, which can be done by comparing event timestamps or sequence numbers in the data stream.

#### Closing summary
Recovery involves stopping the job safely, identifying the last successful checkpoint, replaying from that point, and updating the checkpoint directory accordingly. Monitoring is crucial to ensure no duplicates are generated during this process.

#### Common mistakes

Not stopping the streaming job properly before making changes.
Overwriting existing metadata in the checkpoint directory.
Failing to monitor for duplicate processing during recovery.

