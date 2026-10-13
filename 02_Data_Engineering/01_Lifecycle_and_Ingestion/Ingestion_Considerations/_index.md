---
id: "02_Data_Engineering/01_Lifecycle_and_Ingestion/Ingestion_Considerations"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Ingestion Engineering Considerations

**L4 Advanced** · [↑ Data Engineering Lifecycle and Ingestion](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Before building any ingestion path, evaluate the source system and then decide on boundedness, frequency, synchronicity, serialisation, throughput, reliability, payload and push/pull — because ingestion is where most data pipelines break.

### 1. The big picture

Source systems and ingestion are usually the biggest bottlenecks in the data engineering lifecycle. Sources are outside your control, may become unresponsive, and may produce poor-quality data; ingestion services stop for mysterious reasons. When they fail, storage, transformation and serving all starve, so unreliable ingestion has a ripple effect through the whole lifecycle.

The antidote is to ask the right questions *first*: about the source (what it is, how it persists and generates data, how it changes) and about the ingestion design (how often, how big, in what shape, with what guarantees). This lesson collects those questions and explains the design dimensions. It connects upward to the lifecycle and downward to the batch and stream ingestion lessons, which implement the choices.

### 2. Core concepts and vocabulary

- **Source system** — the application, device fleet, queue or third-party API that produces data.
- **Bounded vs unbounded data** — unbounded data flows continuously as events happen; bounded data is a slice cut by some boundary such as time. "All data is unbounded until it is bounded."
- **Ingestion frequency** — batch, micro-batch or near real time; "real-time" is always near real time because every pipeline has latency.
- **Synchronous ingestion** — source, ingestion and destination steps are tightly coupled; a failure in one stops the rest.
- **Asynchronous ingestion** — stages operate at the level of individual events, buffered by a queue or stream.
- **Serialisation / deserialisation** — encoding data into a transferable format (JSON, Avro, Parquet) and decoding it at the destination.
- **Throughput and elastic scalability** — the volume the system handles, and its ability to absorb spikes.
- **Reliability** — high uptime and failover; **durability** — data isn't lost or corrupted.
- **Payload** — the data actually carried: kind, shape, size, schema, metadata.
- **Push / pull / poll** — source sends; ingester fetches; ingester periodically checks for new data.
- **Schemaless vs fixed schema** — "schemaless" means the schema is defined by the writer on write, not absent.
- **Bastion host / SSH tunnel** — an exposed, locked-down intermediate host used to reach a private database.

### 3. How it works, step by step

**A. Evaluate the source** (questions to answer before design):

1. What is it — an application database, IoT fleet, message queue, third-party API?
2. How is data persisted: long-term, or deleted quickly (so you must catch it in time)?
3. What is the generation rate (events/sec, GB/hour)?
4. How consistent is the output (nulls, bad formats)? How frequent are errors?
5. Are there duplicates? Can data arrive late — possibly much later than its peers?
6. What is the schema, and do you need joins across tables or systems?
7. How are schema changes made and communicated?
8. How often should you read? Will reading hurt the source's performance?
9. For stateful systems: snapshots or CDC events? How are changes tracked?
10. Who is the data provider, and what upstream dependencies does the source have?
11. Are there checks for late or missing data?

**B. Decide the ingestion shape** (questions about your own design):

1. Use case: can I reuse this data instead of ingesting another copy?
2. Destination, update frequency, expected volume.
3. Format: can downstream storage and transformation accept it?
4. Quality: is the data usable immediately? What post-processing and what risks (e.g., bot traffic)?
5. For streams: does it need in-flight transformation?

**C. Walk the design dimensions:**

| Dimension | Question | Typical choices |
|---|---|---|
| Bounded vs unbounded | Do I preserve continuous flow or cut batches? | Streaming preserves unboundedness; batch imposes boundaries |
| Frequency | How fresh must data be? | Daily batch → hourly → micro-batch → streaming |
| Sync vs async | Can stages fail independently? | Prefer async with a buffer (stream/queue) |
| Serialisation | Will the destination parse it? | Avro/Parquet for typed, JSON for flexible |
| Throughput | Can it scale for peaks? | Elastic buffer, partitioned consumers |
| Reliability / durability | What does loss cost? | Redundancy sized to impact |
| Payload | How big, what shape, what metadata? | Compression, schema registry |
| Push/pull/poll | Who initiates? | CDC push, scheduled pull, API polling |

### 4. Worked example

An IoT use case: delivery vans send GPS pings. Connectivity is intermittent; vans upload when they reach depot WiFi.

```text
Source evaluation
  type            IoT gateway on vehicle
  persistence     device keeps 24h buffer, then overwrites         -> must ingest within a day
  rate            1 ping / 5s / van * 2,000 vans = 400 events/s
  late data       YES: hours late after dead zones                  -> event-time handling required
  duplicates      YES: gateway retries on upload                    -> dedup key (van_id, ts)
  schema change   firmware updates add fields                       -> versioned schema, tolerate unknown fields

Ingestion design
  boundedness     unbounded events, landed as 5-min bounded files
  frequency       near real time for live map; batch (hourly) for analytics
  sync/async      async: gateway -> stream buffer -> consumers
  serialisation   Avro with schema registry
  reliability     durable stream with 7-day retention (replay after consumer bugs)
  push/pull       push from device, pull by consumers
```

Reading it: the *source* facts (device keeps only 24 hours, uploads arrive late, retries create duplicates) dictate the *design* (durable buffer, event-time processing, dedup key). Skipping the evaluation would produce a pipeline that quietly drops or double-counts data.

### 5. Under the hood

**Why async beats sync.** In a synchronous chain, process B cannot start until A finishes and any failure forces a rerun of everything. One organisation had dozens of tightly coupled steps running over 24 hours; any failure restarted the whole chain, and cryptic errors made diagnosis take a week. In an async design, a stream (e.g., a managed stream feeding a processing framework that writes to object storage) acts as a shock absorber: producers keep writing when consumers lag, each stage scales to available resources, and each event is available as soon as it passes a stage.

**Frequency is a bottleneck chain.** Even with streaming ingestion, downstream batch jobs are common (ML training, daily marts). The batch frequency then caps freshness for everything after it — you choose *where* the batch boundaries sit, you rarely eliminate them.

**Reliability and durability link.** If a source does not retain data (IoT, caches), ingestion unreliability becomes permanent data loss. If data is ingested, downstream failures merely delay processing. So size redundancy by the cost of loss, including direct cost (infrastructure, on-call) and indirect cost (team strain); no system can survive every scenario, and chasing perfect durability has diminishing returns.

**Shell and SSH.** Simple ingestion often starts as a shell script: dump from a database, reserialise, upload via a cloud CLI, trigger a load. It works for modest sources; as SLAs tighten and steps multiply, move to orchestration. SSH is a transport, not a strategy: used for SCP file transfer and for tunnels through a bastion host so application databases are never directly exposed to the internet.

### 6. Performance and design implications

- **Source impact**: heavy extraction competes with application traffic; read replicas, log-based CDC, or off-peak windows reduce it.
- **Backpressure**: a buffer decouples producer and consumer rates; measure consumer lag.
- **Throughput headroom**: provision for peak or use autoscaling; test at 2–3× normal volume.
- **Format cost**: text formats are easy but large and slow to parse; binary typed formats shrink payload and enforce schema.
- **Monitor**: freshness (now − latest event time), completeness (expected vs received counts), duplicate rate, schema-change events, error and retry rates.
- **Data quality at the edge**: validate on landing and quarantine bad records rather than failing the whole batch.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `ssh -L <local>:<db-host>:<port> user@bastion` | Opens a tunnel through a bastion host | Reaching private databases securely |
| `scp` / `sftp` | Secure file transfer | Receiving partner or exported files |
| `aws s3 cp` / equivalent cloud CLI | Uploads/downloads objects | Shell-based ingestion |
| Schema registry | Stores and versions message schemas | Streaming with evolving payloads |
| Watermark / checkpoint store | Tracks progress per source | Safe resume and idempotent reruns |
| Dead-letter queue / quarantine table | Holds records that fail validation | Keeping pipeline flowing while isolating bad data |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Streaming ingestion — events needed within seconds, source is event-native | Consumers only need daily data; complexity outweighs benefit |
| Micro-batch — near-real-time with simpler semantics | Strict per-event latency required |
| Async with buffer — unreliable consumers, spiky producers | Strictly ordered transactional workflows with tiny volume |
| Sync batch — simple, small, well-bounded | Long chains where failure forces total reruns |
| Pull — you control schedule and the source is stable | Source cannot tolerate additional queries |
| Push / CDC — need low latency, low source load | Source team can't or won't emit events |
| Maximum redundancy — data irrecoverable once lost | Data is replayable; cost outweighs benefit |

### 9. Common mistakes

- **Skipping source evaluation** → surprises about late data, duplicates, schema drift → run the checklist first.
- **Tightly coupled synchronous chain** → one failure reruns everything → decouple with buffered stages and idempotent steps.
- **Assuming "real-time" is zero latency** → missed SLAs → state latency targets in numbers.
- **Querying the production database directly for analytics** → performance hit and security exposure → replica, CDC or file export.
- **Exposing the application database to the internet** → security incident → bastion host and SSH tunnel or private networking.
- **Ignoring source retention** → permanent loss of data when ingestion is down → alert on lag well within the retention window.
- **Overbuilding durability** → cost and on-call burden with little value → match to the impact of loss.

### 10. 🎯 Interview kit

**30-second answer:** "Ingestion is the most failure-prone stage because sources are outside our control. I first profile the source — persistence, rate, consistency, duplicates, lateness, schema changes, load impact — then choose boundedness, frequency, sync vs async, serialisation, scaling and durability to fit, preferring buffered async designs with idempotent, monitored steps."

**Structure a longer answer:** clarify use case, volume and freshness → mechanism (source evaluation, then design dimensions) → trade-offs (batch vs stream, push vs pull, cost of durability) → real scenario (e.g., the IoT example).

**Likely follow-ups:**
- *What does "all data is unbounded until it's bounded" mean?* → Data originates as an ongoing event flow; batches are artificial boundaries.
- *Why is async better?* → Failures are isolated, stages scale independently, the buffer absorbs spikes.
- *How do you handle late-arriving data?* → Event-time processing, watermarks/allowed lateness, reprocess partitions.
- *How do you ingest from a database without hurting it?* → Replica, log-based CDC, off-peak windows, incremental reads.

**Signals interviewers look for:** structured questioning of the source, awareness of failure modes, quantitative thinking (rates, retention), and sensible cost/risk calibration.

### 11. Check yourself

- [ ] List eight questions you would ask about a new source system.
- [ ] Explain bounded vs unbounded data and why batches are artificial.
- [ ] Contrast synchronous and asynchronous ingestion with a failure example.
- [ ] Describe how a bastion host and SSH tunnel protect a database.
- [ ] Explain the link between reliability of ingestion and durability of data.
- [ ] Design an ingestion plan for a source that retains only 24 hours of data and sends duplicates.
<!-- explainer:end -->

## References

- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Ingestion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Key Engineering Considerations for the Ingestion Phase
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Key engineering considerations for the ingestion phase
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Reliability and Durability
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Frequency
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Synchronous Versus Asynchronous Ingestion
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Evaluating source systems: Key engineering considerations
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Selecting a storage system
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Ingestion › Shell
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — Designing Good Data Architecture › Devices

