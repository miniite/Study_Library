---
id: "03_Big_Data_and_Streaming/01_Batch_Processing/MapReduce_Model"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# The MapReduce Programming Model

**L4 Advanced** · [↑ Batch Processing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** MapReduce expresses a computation as a pure *map* function that turns each record into key-value pairs and a pure *reduce* function that processes all values for one key, with the framework handling partitioning, sorting, movement and retries.

### 1. The big picture

MapReduce is a deliberately restricted programming model for large-scale processing on a cluster. You write two callbacks; the framework splits the input, runs mappers near the data, groups and sorts intermediate pairs by key, feeds them to reducers, and re-runs anything that fails. The restriction (pure functions over immutable input) is precisely what allows it to run anywhere, in any order, and be retried.

It is rarely used directly today, but it defines vocabulary used by Spark, Hive, Pig and Cascading, and its constraints explain why those systems look the way they do. This lesson covers the model and type signatures, the combiner optimisation, the idea of workflows (many jobs), and the higher-level abstractions built on top of it. *MapReduce Job Anatomy* covers the execution machinery.

### 2. Core concepts and vocabulary

- **Mapper** — called once per input record; extracts zero or more `(key, value)` pairs; keeps no state between records.
- **Reducer** — called once per distinct key with an iterator over all values for that key; emits output records.
- **Pure function** — uses only its inputs, has no side effects (no external queries), so it can be rerun anywhere.
- **Key-value types** — `map: (K1, V1) → list(K2, V2)`; `reduce: (K2, list(V2)) → list(K3, V3)`.
- **Writable types** — Hadoop's serialisation-optimised wrappers (`Text`, `IntWritable`, `LongWritable`).
- **Combiner** — an optional "mini-reducer" run on map output to shrink data before the shuffle; same signature as reduce but outputs `(K2, V2)`.
- **Algebraic function** — an aggregate computable incrementally from partial results (MAX, SUM, COUNT); required for safe combiner use.
- **Workflow** — a chain of jobs where one job's output is the next job's input.
- **Higher-level abstraction** — Pig, Hive, Cascading, Crunch, Spark that compile to or replace raw MapReduce.

### 3. How it works, step by step

1. Input is split into records by an input format (e.g., each line is a record; the key is the byte offset).
2. **Map**: `map(K1, V1)` emits `(K2, V2)` pairs; discard bad records here.
3. **Combine** (optional): per map task, locally aggregate pairs with the same key.
4. **Partition + sort**: hash of key chooses the reducer; each reducer's input is sorted by key.
5. **Reduce**: `reduce(K2, [V2...])` emits output.
6. **Write** output files to the distributed filesystem.

```text
map:      (K1, V1)         -> list(K2, V2)
combiner: (K2, list(V2))   -> list(K2, V2)
reduce:   (K2, list(V2))   -> list(K3, V3)
```

Counting and sorting needs two jobs: job 1 emits `(word, 1)` and sums; job 2 swaps to `(count, word)` and sorts. The rule of thumb is to add more jobs rather than make single jobs more complex.

### 4. Worked example

Max temperature per year (weather records):

```java
public class MaxTemperatureMapper
    extends Mapper<LongWritable, Text, Text, IntWritable> {
  private static final int MISSING = 9999;
  public void map(LongWritable key, Text value, Context context)
      throws IOException, InterruptedException {
    String line = value.toString();
    String year = line.substring(15, 19);
    int airTemperature = (line.charAt(87) == '+')
        ? Integer.parseInt(line.substring(88, 92))
        : Integer.parseInt(line.substring(87, 92));
    String quality = line.substring(92, 93);
    if (airTemperature != MISSING && quality.matches("[01459]")) {
      context.write(new Text(year), new IntWritable(airTemperature));
    }
  }
}
```

```java
Job job = new Job();
job.setJarByClass(MaxTemperatureWithCombiner.class);
job.setMapperClass(MaxTemperatureMapper.class);
job.setCombinerClass(MaxTemperatureReducer.class);  // same class as the reducer
job.setReducerClass(MaxTemperatureReducer.class);
job.setOutputKeyClass(Text.class);
job.setOutputValueClass(IntWritable.class);
```

For 1950, one map outputs `(1950,0) (1950,20) (1950,10)` and another `(1950,25) (1950,15)`. Without a combiner the reducer sees `[0,20,10,25,15]` → 25. With a max combiner it sees `[20,25]` → 25: same answer, less data shuffled. Using a *mean* as the combiner would break correctness: `mean(0,20,10,25,15)=14` but `mean(mean(0,20,10), mean(25,15)) = mean(10,20) = 15`.

The same aggregation in Spark: `pairs.reduceByKey(_ + _)`, where the function must be commutative and associative so partial aggregation per partition is valid.

### 5. Under the hood

**Contract of the combiner.** Hadoop does not guarantee how many times (zero, one, many) a combiner runs on any map output. Therefore correct results must not depend on it: `f(f(a,b), c) = f(a, f(b,c))` and the output type must equal the intermediate type. It reduces bandwidth, often the bottleneck.

**Type system.** Map output types must equal reduce input types, but Java generics do not enforce this; mismatches show up at runtime. The partitioner, sort comparator and grouping comparator are further pluggable pieces over `K2`.

**Why pure functions matter.** Because mapper and reducer cannot query external stores or mutate state, the framework can run them on any node in any order and rerun on failure; the same restriction exists for MongoDB's map-reduce, where JavaScript functions must be pure. A usability downside: two coordinated functions are harder to write than a declarative query, and an optimiser has little to work with — one reason MongoDB added the aggregation pipeline and Hadoop users moved to Hive/Pig. NoSQL systems tend to "reinvent SQL in disguise".

**Higher-level layers.** Cascading replaces raw key/values with named fields and tuples, and pipes (`Each`, `GroupBy`, `CoGroup`, `Every`); a planner partitions the pipe assembly into MapReduce jobs and links them. Pig has eval, filter, load and store functions; algebraic eval functions (MAX) use combiners while non-algebraic ones (median) cannot. HBase integrates via `TableInputFormat` (splits on region boundaries) and `TableOutputFormat`.

### 6. Performance and design implications

- **Shuffle volume is the cost driver**: filter early in map, emit small values, use combiners for algebraic aggregates.
- **Skew**: one hot key = one slow reducer (see *Skew Handling*).
- **Tasks**: too many tiny map tasks waste start-up time; too few limit parallelism. Reducer count is set by you; map count follows input splits.
- **Memory**: reducers iterate values lazily; do not buffer an entire group unless you must.
- **Scalability**: linear scaling with data and cluster size, assuming balanced partitions.
- **Use higher-level tools** (Hive, Pig, Spark) for joins and multi-stage logic to avoid hand-written joins.

### 7. Commands and settings you should know

| Command / setting | What it does | When to use |
|---|---|---|
| `job.setCombinerClass(...)` | Enables map-side partial aggregation | Algebraic aggregates (sum, max, count) |
| `job.setNumReduceTasks(n)` | Sets reduce parallelism (0 = map-only) | Tuning output files and shuffle |
| `job.setPartitionerClass(...)` | Chooses which reducer receives a key | Custom partitioning, skew |
| `job.setInputFormatClass / setOutputFormatClass` | Chooses formats (text, sequence, Avro, Parquet) | Reading and writing data |
| `hadoop jar` | Runs the job | Submission |
| Counters | Count events in tasks (e.g., bad records) | Data quality monitoring |

### 8. Trade-offs and decisions

| Choose this when | Avoid / watch out when |
|---|---|
| Learning distributed batch concepts | Starting new production work (use Spark/Hive/warehouse) |
| Embarrassingly parallel scans and aggregations | Iterative algorithms (each iteration re-reads from disk) |
| Aggregates that are algebraic (use combiners) | Aggregates like median, distinct count exact, mean-of-means |
| Multiple small jobs chained | One giant job with complicated logic |

### 9. Common mistakes

- Using a non-associative function (mean) as a combiner → wrong results that vary with execution → carry `(sum, count)` pairs.
- Emitting huge values per key → memory and network blow-ups → pre-aggregate or restructure the key.
- Relying on the combiner always running → logic bugs → treat it as an optimisation only.
- Doing external lookups in `map()` → non-determinism and overload → join in the framework (see joins lesson).
- Hand-coding multi-way joins → complexity and bugs → use Pig/Hive/Spark.

### 10. 🎯 Interview kit

**30-second answer:** MapReduce is a two-function model: map turns each record into key-value pairs; the framework groups and sorts by key; reduce handles each key's values. Functions are pure so tasks can be retried and run anywhere. A combiner pre-aggregates map output to cut shuffle traffic, but must be associative and commutative.

**Structure a longer answer:** signature → example (word count / max temperature) → shuffle and combiner → restrictions and why → limitations and modern alternatives.

**Likely follow-ups:**
- *Why can't mean be a combiner?* → Mean of partial means ≠ overall mean; carry sum and count.
- *How many mappers/reducers?* → Mappers by input splits; reducers by configuration.
- *How do you do a sort in MapReduce?* → Second job, or total-order partitioner.
- *Why pure functions?* → Allows re-execution and arbitrary placement.

**Signals interviewers look for:** correct combiner reasoning, awareness of shuffle cost, and the ability to reformulate problems as chained jobs.

### 11. Check yourself

- [ ] Write map and reduce signatures for word count and a top-N-by-count follow-up.
- [ ] Demonstrate with numbers why `mean` is an invalid combiner.
- [ ] Explain how the reducer receives a sorted iterator and why that bounds memory.
- [ ] State what Cascading and Pig add over raw MapReduce.
- [ ] Describe when the number of reducers should be zero.
<!-- explainer:end -->

## In this section (study in order)

1. [Anatomy of a MapReduce Job Run](<MapReduce_Job_Anatomy/_index.md>) (L5 Expert)
2. [Shuffle and Sort Internals](<Shuffle_and_Sort/_index.md>) (L6 Internals)

## References

- *Hadoop: The Definitive Guide* — Tom White — Chapter 24. Cascading › Fields, Tuples, and Pipes
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Combiner Functions
- *Hadoop: The Definitive Guide* — Tom White — MapReduce Types and Formats › Chapter 8. MapReduce Types and Formats
- *Hadoop: The Definitive Guide* — Tom White — Cascading in Practice › Flexibility
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Specifying a combiner function
- *Hadoop: The Definitive Guide* — Tom White — Developing a MapReduce Application › MapReduce Workflows
- *Hadoop: The Definitive Guide* — Tom White — MapReduce › Java MapReduce
- *Designing Data-Intensive Applications* — Martin Kleppmann — Data Models and Query Languages › MapReduce Querying
- *Hadoop: The Definitive Guide* — Tom White — Meet Hadoop › Comparison with Other Systems
- *Hadoop: The Definitive Guide* — Tom White — HBase › MapReduce

