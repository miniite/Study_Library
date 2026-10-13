---
type: "practice"
difficulty: "Deep"
---

# How would you build a streaming pipeline that computes the rate of new distinct error signatures per minute to catch a novel failure mode? What sketch (HyperLogLog) keeps distinct-count feasible?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Provisional and Final Results](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this requirement, we need to design a robust real-time streaming pipeline that computes the rate of new distinct error signatures per minute. The goal is to detect novel failure modes quickly and accurately. We'll use HyperLogLog (HLL) sketch to make distinct count feasible in real-time streams.

The core idea here is to leverage HLL for approximate distinct counting, which allows us to process large volumes of data efficiently without consuming excessive memory. This solution involves several steps: stream processing, error signature extraction, and aggregation using HLL sketches.

#### Overview
We'll design a streaming pipeline that ingests logs in real-time, extracts error signatures from log messages, and uses HyperLogLog sketches for approximate distinct count to compute the rate of new distinct error signatures per minute.

#### Triage
The question is probing our understanding of real-time stream processing architectures, approximation algorithms like HLL, and how to handle large-scale data in a streaming context. We'll assume that logs are ingested via a reliable messaging system such as Apache Kafka or AWS Kinesis. Our focus will be on the distinct count problem and its approximate solution.

#### First-Principles Solution
Log Ingestion: Use a real-time stream processing framework like Apache Flink or Apache Spark Streaming to ingest logs from a message broker (e.g., Kafka).
Error Signature Extraction: Parse each log message to extract relevant fields, such as the error code and timestamp. Group messages by error signature.
HLL Sketching: Use HLL sketches to maintain an approximate count of distinct error signatures per minute window.

#### Closing Summary
+ Common Mistakes/Follow-Ups: This approach ensures that we can handle a high volume of logs in real-time while providing accurate estimates of the number of unique error signatures. Key points include the use of HLL for memory efficiency and the importance of choosing an appropriate time window for distinct count.

