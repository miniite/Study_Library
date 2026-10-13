---
id: "TOOLS/Hadoop/02_YARN_and_MapReduce/MapReduce_Development"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# Developing MapReduce Applications

**L4 Advanced** · [↑ YARN and MapReduce on Hadoop](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Developing a MapReduce application is a loop of writing small, testable map and reduce functions, running a driver locally on a sample, then scaling to the cluster while using counters, the right data types/formats, compression, combiners and joins to make it correct and fast.

### 1. The big picture
The programming model is small (map, shuffle/sort, reduce), but real applications need engineering around it: configuration, a driver, unit tests, data types and input/output formats, side channels (counters, side data), joins, and tuning. The workflow the book recommends is incremental: write the functions with unit tests → run the driver from the IDE on a small subset → fix with the debugger and extend the tests → run on the cluster over the full dataset (new problems appear; repeat) → tune.

This sits on top of the YARN/MapReduce overview (how a job is run) and next to the failures topic (how it survives). It is also the foundation for understanding Hive, Pig and Crunch, which generate this kind of job for you.

### 2. Core concepts and vocabulary
- **Driver** — the program that configures and submits a `Job`; usually implements `Tool` and is launched with `ToolRunner` so generic `-D` options work.
- **Mapper / Reducer** — `map(K1,V1)→(K2,V2)*`; `reduce(K2,[V2])→(K3,V3)*`.
- **Combiner** — a mini-reducer run on map output to cut shuffle volume; must be commutative/associative and its input and output types must match.
- **Partitioner** — decides which reducer gets a key (default: hash of key).
- **InputFormat / OutputFormat** — define how input is split and parsed and how output is written (`TextInputFormat`, `SequenceFile…`, `MapFileOutputFormat`, …).
- **Counter** — a named, globally aggregated number updated by tasks.
- **Side data** — read-only extra data distributed to tasks (small lookup files via the distributed cache).
- **Map-side / reduce-side join** — a join done in the mapper or in the reducer.
- **Secondary sort** — control the order of *values* seen by a reducer.
- **MultipleOutputs / MultipleInputs** — several output files per reducer / several inputs with different mappers.

### 3. How it works, step by step
1. **Configure types.** The job declares the five type families: input format (K1,V1), map output (K2,V2) via `setMapOutputKeyClass/ValueClass`, final output (K3,V3) via `setOutputKeyClass/ValueClass`, plus mapper, combiner, partitioner, sort and grouping comparators, reducer and output format.
2. **Test the functions.** Unit-test mapper and reducer in isolation (e.g., MRUnit) with a few records, including malformed ones.
3. **Run locally.** Use local job runner mode (`mapreduce.framework.name=local`, `fs.defaultFS=file:///`) from the IDE against a small sample; debug with breakpoints.
4. **Run on the cluster.** Package a jar, `hadoop jar … -D key=value`. Watch the RM web UI; read task logs; use counters to see data-quality statistics.
5. **Tune.** Check the standard list (below), then profile tasks if needed.

**Reduce-side join, step by step**
1. Use `MultipleInputs` with one mapper per source to parse each format.
2. Each mapper emits a composite key `(joinKey, tag)` where the tag orders sources (stations tagged 0, weather records 1).
3. A partitioner and grouping comparator send all tags of one `joinKey` to the same reducer but order the values by tag (secondary sort).
4. The reducer sees the small-side record first, remembers it, and emits it joined with every following record — no buffering of the large side.

### 4. Worked example
```java
public class MaxTemperatureWithCounters extends Configured implements Tool {
  enum Temperature { MISSING, MALFORMED }

  static class M extends Mapper<LongWritable, Text, Text, IntWritable> {
    private NcdcRecordParser parser = new NcdcRecordParser();
    @Override protected void map(LongWritable k, Text v, Context ctx)
        throws IOException, InterruptedException {
      parser.parse(v);
      if (parser.isValidTemperature()) {
        ctx.write(new Text(parser.getYear()), new IntWritable(parser.getAirTemperature()));
      } else if (parser.isMalformedTemperature()) {
        ctx.getCounter(Temperature.MALFORMED).increment(1);
      } else if (parser.isMissingTemperature()) {
        ctx.getCounter(Temperature.MISSING).increment(1);
      }
      ctx.getCounter("TemperatureQuality", parser.getQuality()).increment(1); // dynamic counter
    }
  }

  public int run(String[] args) throws Exception {
    Job job = JobBuilder.parseInputAndOutput(this, getConf(), args);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    job.setMapperClass(M.class);
    job.setCombinerClass(MaxTemperatureReducer.class);   // max is associative
    job.setReducerClass(MaxTemperatureReducer.class);
    return job.waitForCompletion(true) ? 0 : 1;
  }
  public static void main(String[] a) throws Exception {
    System.exit(ToolRunner.run(new MaxTemperatureWithCounters(), a));
  }
}
```
At the end the framework prints each counter group (the enum name is the group, the fields are the counters) aggregated over all map and reduce tasks. A huge `MALFORMED` count tells you the parser or data is wrong — far cheaper than grepping logs from thousands of tasks. Because maximum is commutative and associative, the same class is safe as combiner and reducer.

### 5. Under the hood
- **Types contract:** only `K2/V2` matter between map and reduce; a combiner must emit the same types it receives. Reduce *input* keys are guaranteed sorted, but reduce *output* keys have no ordering contract — `MapFileOutputFormat` requires sorted keys, so the reducer must emit in order.
- **Output files:** one per reducer, named `part-r-00000…`; `MultipleOutputs` allows custom naming and several files per reducer. With zero reducers the job is map-only and output files are `part-m-*`.
- **Counters vs logs:** counters are aggregated by the framework and available through the UI, API and command line; they are the right tool when you are tempted to log in a task. Use enums for fixed sets and dynamic `getCounter(group, name)` for data-driven names; counter counts are limited, so don't create one per record.
- **Compressing map output:** set `mapreduce.map.output.compress=true` and `mapreduce.map.output.compress.codec` to a fast codec (LZO, LZ4, Snappy). It reduces disk and network volume in the shuffle even if input and output are uncompressed.
- **Map-side join** requires that one dataset fits in memory on each node (distributed cache), or that both are sorted and identically partitioned. **Reduce-side join** needs no structure but pays full shuffle cost for both inputs.
- Long-running tasks that do not read input (e.g., simulations) must report progress; bootstrap-style jobs (seed file → each mapper loads one source) set reducers to zero.

### 6. Performance and design implications
Standard checks before profiling:
1. Number of mappers — avoid thousands of tiny splits; combine small files.
2. Number of reducers — enough to use the cluster but not so many that output is fragmented; match to available containers.
3. Combiners — use them when aggregation allows.
4. Intermediate compression — almost always a win.
5. Custom serialization — use compact `Writable`s or Avro rather than text.
6. Shuffle tuning — buffer sizes, merge factor, number of parallel copies.
Prefer higher-level tools (Hive, Pig, Crunch, Spark) for joins: hand-written joins are involved and error-prone.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `hadoop jar app.jar Driver -D k=v` | Run with generic options (`ToolRunner`) | Every run |
| `-conf`, `-fs`, `-jt`, `-files`, `-libjars` | Generic options | Local vs cluster, side data |
| `mapreduce.framework.name=local` | Local job runner | Debugging |
| `job.setCombinerClass` | Mini-reduce on map side | Aggregations |
| `mapreduce.map.output.compress(.codec)` | Compress shuffle data | Large shuffles |
| `MultipleInputs`, `MultipleOutputs` | Many sources / outputs | Joins, partitioned output |
| `mapred job -counter <id> <group> <name>` | Read a counter | Data-quality checks |
| `job.setNumReduceTasks(0)` | Map-only job | Loading, filtering |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Map-side join with small side data in the distributed cache | Side data too large for each task's memory |
| Reduce-side join for two large, unstructured inputs | Latency-sensitive jobs (full shuffle of both) |
| Combiner | Non-associative aggregations (e.g., mean of means) |
| Counters for quality metrics | Using counters as a data store (high cardinality) |
| Hive/Pig/Spark for joins and ad hoc work | Needing custom partitioning/IO that they can't express |

### 9. Common mistakes
- A combiner that changes output types or isn't associative → wrong results that vary with spilling → test results with and without the combiner.
- Forgetting to set map output types when they differ from final output → runtime type mismatch → declare `setMapOutputKeyClass/ValueClass`.
- Buffering all values for a key in the reducer → out of memory → secondary sort.
- Reusing a `Writable` object while storing a reference to it → corrupted data → copy values you keep.
- Logging instead of counting → unsearchable logs → counters.
- Debugging only on the full cluster → slow loop → local runner and unit tests first.

### 10. 🎯 Interview kit
**30-second answer:** Write mapper and reducer with unit tests, run through a `Tool`-based driver locally then on the cluster, use combiners and map-output compression to cut shuffle cost, counters for data quality, and choose map-side or reduce-side joins by dataset size.

**Structure a longer answer:** clarify dataset sizes and shape → data flow with types → optimisations (combiner, compression, reducers) → joins/side data → how you would test and debug.

**Likely follow-ups:**
- *What conditions must a combiner satisfy?* → associative and commutative; same input/output types; may run zero or more times.
- *Map-side vs reduce-side join?* → in-memory small side vs full shuffle of both.
- *How do you make the reducer see the lookup record first?* → composite key with tag + secondary sort.
- *How do you count bad records?* → counters.
- *How do you reduce shuffle volume?* → combiner, compression, filtering early, compact types.

**Signals interviewers look for:** correct reasoning about types, associativity and memory limits; knowledge of alternatives.

### 11. Check yourself
- [ ] Write a mapper/reducer pair and its driver using `ToolRunner`.
- [ ] Explain why a combiner for an average needs a different design than for a maximum.
- [ ] Sketch a reduce-side join with tags and secondary sort.
- [ ] Enable map-output compression with Snappy.
- [ ] List the tuning checklist and what each item buys you.
<!-- explainer:end -->

## References

- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › Chapter 8. MapReduce Types and Formats
- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › Chapter 6. Developing a MapReduce Application
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Reduce-Side Joins
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › MapFileOutputFormat
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Chapter 9. MapReduce Features
- *Hadoop: The Definitive Guide* — Tom White — K › M
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › Joins
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Compressing map output
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Features › User-Defined Java Counters
- *Hadoop: The Definitive Guide* — Tom White — How MapReduce Works › The Reduce Side

