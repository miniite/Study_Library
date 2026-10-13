---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/MapReduce_Model/MapReduce_Job_Anatomy"
level: 5
type: "curriculum"
curriculum: "data_engineering"
---

# Anatomy of a MapReduce Job Run

**L5 Expert** · [↑ The MapReduce Programming Model](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A Hadoop MapReduce job run involves a client, the YARN resource manager and node managers, a per-job application master, and HDFS: the client computes splits and submits, the application master schedules map and reduce tasks in containers, and progress, counters and failures flow back through the umbilical interface until the job commits its output.

### 1. The big picture

*MapReduce Model* explains what you write; this lesson explains what Hadoop *does* when you call `job.waitForCompletion(true)`. Knowing the machinery lets you read job IDs, counters and the web UI, understand why the output directory must not exist, choose reducer counts, and see why multi-job workflows are slow because every stage's output is materialised on the distributed filesystem.

The anatomy also exposes MapReduce's central cost: job-to-job communication goes only through files in HDFS, so a workflow of 50–100 jobs (as in recommendation pipelines) writes enormous intermediate state.

### 2. Core concepts and vocabulary

- **Client** — the program that submits the job.
- **Resource manager (RM)** — YARN daemon that allocates cluster resources.
- **Node manager (NM)** — per-machine daemon that launches and monitors containers.
- **Application master (AM)** — per-job coordinator; runs in a container; schedules tasks, tracks progress.
- **Container** — allocation of CPU/memory in which a task JVM runs.
- **Input split** — chunk of input processed by one map task (by default one HDFS block).
- **Task attempt** — one execution of a task; failed or killed attempts are retried as new attempts.
- **Job / task / attempt ID** — e.g., `job_local26392882_0001`, `attempt_..._m_000000_0` (`m` = map, `r` = reduce).
- **Umbilical interface** — channel through which a task reports progress/status/counters to its AM.
- **Counters** — framework-defined and user-defined event counts per job.
- **Output committer** — component that promotes task output to the final directory and cleans up.
- **Materialisation** — writing intermediate job output to the distributed filesystem.

### 3. How it works, step by step

![How Hadoop runs a MapReduce job](<../../../../_assets/Hadoop_The_Definitive_Guide_Tom_White/hddg_0701.png>)

**Submission (client side)**

1. `Job.submit()` creates a `JobSubmitter`; `waitForCompletion()` then polls once per second.
2. Asks the RM for a new application ID (becomes the job ID).
3. Checks the output specification: if the output directory is unset or *already exists*, the job fails fast, preventing accidental overwrites.
4. Computes input splits; fails if input paths don't exist.
5. Copies the job JAR (replication factor 10 by default, `mapreduce.client.submit.file.replication`), configuration and split info to a directory named after the job ID on HDFS.
6. Calls `submitApplication()` on the RM.

**Execution**

7. The RM starts a container for the application master on some node manager.
8. The AM requests containers for map tasks (one per split, preferring nodes that hold the block — data locality) and reduce tasks (count set by the job author).
9. Each task runs in its own container/JVM; the NM launches it.
10. Map tasks read splits via the input format, call `map()`, partition and sort output; reducers fetch their partitions, merge-sort and call `reduce()`.

**Progress and completion**

11. Tasks report progress and counters to the AM every three seconds via the umbilical interface; the client polls the AM.
12. When the last task finishes, the AM marks the job successful; the client prints counters; optional HTTP end-notification fires (`mapreduce.job.end-notification.url`).
13. The AM and containers clean up intermediate output; `OutputCommitter.commitJob()` runs; the history server archives job info.

### 4. Worked example

```java
Job job = new Job();
job.setJarByClass(MaxTemperature.class);            // locates the JAR to distribute
FileInputFormat.addInputPath(job, new Path(args[0]));   // may be file, dir or glob; callable repeatedly
FileOutputFormat.setOutputPath(job, new Path(args[1])); // must NOT exist
job.setMapperClass(MaxTemperatureMapper.class);
job.setReducerClass(MaxTemperatureReducer.class);
job.setOutputKeyClass(Text.class);
job.setOutputValueClass(IntWritable.class);
System.exit(job.waitForCompletion(true) ? 0 : 1);   // exit code 0 = success
```

```bash
hadoop jar hadoop-examples.jar MaxTemperature input/ncdc/sample.txt output
cat output/part-r-00000
# 1949  111      (11.1°C — values stored ×10)
# 1950  22
```

Reading the console: job ID `job_local26392882_0001`, one map attempt (`..._m_000000_0`) and one reduce attempt (`..._r_000000_0`). Counters show *Map input records = 5*, *Map output records = 5*, *Reduce input groups = 2*, *Reduce output records = 2* — a sanity check that all records flowed through and that two distinct keys (years) were grouped. The output has one file per reducer (`part-r-NNNNN`; `m` for map-only jobs).

Minimal defaults, spelled out: input format `TextInputFormat` (key = `LongWritable` byte offset, value = `Text` line), mapper = identity `Mapper`, partitioner = `HashPartitioner`, one reducer, output `TextOutputFormat`.

### 5. Under the hood

**Defaults worth knowing.**

- `HashPartitioner.getPartition = (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks` — masks the sign bit, then modulo; same key → same reducer.
- The number of map tasks equals the number of input splits (file size and block size); you do not set it directly.
- The default single reducer is a gotcha: the entire shuffle funnels through one task. Real jobs nearly always set a higher count (a balance between parallelism, many small output files and per-task overhead).

**Progress accounting.** Map progress = fraction of input consumed. Reduce progress is split into thirds for the three shuffle phases (copy, sort, reduce); half-way through reduce means 5/6 complete. Operations that count as progress (so Hadoop won't time out the task): reading an input record, writing an output record, setting a status, incrementing a counter, or calling `progress()`. A task silently stuck in a long computation without any of these can be killed as hung.

**Task status and attempts.** Attempts can be *failed* (exception, timeout, JVM crash) or *killed* (speculative duplicate lost the race, node died, user kill). The AM retries failed tasks on other nodes; repeated failures fail the job. Speculative execution launches a duplicate of a slow task; whichever finishes first wins.

**Workflows and job chaining.** Each job's only contact with the world is its input and output directories. A linear chain can be run sequentially with `waitForCompletion()` (if a job fails, don't run the next; consider cleaning intermediate data). A DAG of jobs needs `JobControl` (dependency graph run in a thread; if a job fails its dependents don't run) or an external scheduler (Oozie, Airflow). Outputs published for many consumers belong in well-known named locations; outputs used by one successor are merely *intermediate state* — and because the next job can only start when *all* tasks of the previous have finished, stragglers delay the whole workflow, and redundant mappers just re-read what reducers wrote.

### 6. Performance and design implications

- **Split size/data locality**: tasks scheduled on nodes holding the block avoid network reads. Many tiny files = many tiny tasks = overhead dominated by container start-up.
- **Reducers**: choose so each reducer processes a reasonable data volume (hundreds of MB to a few GB); too few → slow; too many → many small files and overhead.
- **Combiners and compression** of map output reduce shuffle I/O.
- **Memory**: the container memory settings for map/reduce must cover the task heap plus buffers (e.g., Parquet writers).
- **Counters** are the cheap way to validate data volumes; use user counters for bad-record counts.
- **Intermediate materialisation** is the dominating cost for long chains: prefer fewer, larger jobs or move to a dataflow engine.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `hadoop jar <jar> <class> <in> <out>` | Submit a job | Running jobs |
| `mapred job -status / -list / -kill <jobId>` | Inspect and control jobs | Operations |
| `yarn application -list / -kill / -status` | Cluster-level app control | Stuck or runaway jobs |
| `yarn logs -applicationId <id>` | Fetch task logs after completion | Debugging failures |
| `mapreduce.job.reduces` | Number of reducers | Tuning parallelism |
| `mapreduce.map.memory.mb`, `mapreduce.reduce.memory.mb` | Container memory | Fixing OOM kills |
| `mapreduce.task.timeout` | Kill a task with no progress after this period | Detecting hangs |
| `mapreduce.map.speculative` / `reduce.speculative` | Speculative execution | Straggler mitigation |
| `mapreduce.job.end-notification.url` | HTTP callback on completion | Integration with schedulers |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| `JobControl`/scheduler for DAGs of jobs | Hand-rolled shell chaining without failure handling |
| Many reducers for big shuffles | Output file count explodes (small-file problem) |
| Speculative execution on heterogeneous clusters | Tasks with external side effects (duplicates!) |
| Materialising published datasets with stable names | Treating every intermediate as a public dataset |
| Dataflow engine for long chains | Single simple job where overhead is tolerable |

### 9. Common mistakes

- Reusing an existing output directory → job refuses to start → write to a new path (by design, to prevent data loss).
- Leaving the default single reducer → hours-long reduce phase → set `mapreduce.job.reduces`.
- Side effects in tasks (writing to external DB) with speculative execution → duplicate writes → disable speculation or make writes idempotent.
- Reading task logs only from the client → missing real errors → use the application master UI/`yarn logs`.
- Ignoring counters → silent data loss (e.g., malformed records dropped) → count and assert on them.

### 10. 🎯 Interview kit

**30-second answer:** The client computes input splits, validates that the output directory doesn't exist, uploads the JAR/config/splits to HDFS and submits to the YARN resource manager. The RM starts an application master in a container; it requests containers for one map task per split (preferring data-local nodes) and the configured number of reducers, tracks progress via status updates, retries failed attempts, and on completion commits output and cleans up.

**Structure a longer answer:** five actors → submission steps → task scheduling and locality → progress/failure handling → commit; then mention the cost of materialising chained jobs.

**Likely follow-ups:**
- *Why must the output directory not exist?* → Protects previous results from accidental overwrite.
- *How many mappers and reducers?* → Mappers = splits; reducers = configured (default 1).
- *What is a killed vs failed attempt?* → Killed: speculative loser, dead node or user; failed: error or timeout.
- *How does Hadoop know a task is alive?* → Progress/status/counter updates via the umbilical interface.

**Signals interviewers look for:** correct role of YARN components, awareness of locality and speculation, and practical debugging knowledge (counters, IDs, logs).

### 11. Check yourself

- [ ] Name the five entities in a MapReduce run and what each does.
- [ ] List what `JobSubmitter` does, in order.
- [ ] Explain how reduce progress reaches 5/6.
- [ ] Describe what happens when a map task attempt fails or is killed by speculation.
- [ ] Explain how you would run a DAG of five MapReduce jobs and what limits its speed.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › The MapReduce job page
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Job Submission
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Chapter 7. How MapReduce Works
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Task Execution
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Note
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Progress and Status Updates
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Java MapReduce
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › Job Completion
- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › JobControl
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › The Default MapReduce Job

