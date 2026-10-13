---
id: "TOOLS/Apache_Spark/04_Spark_Data_Lakes_and_ML"
level: 3
type: "curriculum"
curriculum: "data_engineering"
---

# Spark Data Lakes and Machine Learning

**L3 Practitioner** · [↑ Apache Spark](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Spark is not only a transformation engine: it is also the compute layer for reliable lake/lakehouse storage (ACID tables such as Delta Lake) and for distributed machine learning (MLlib pipelines), so the same cluster takes data from raw files to a deployed model.

### 1. The big picture
Writing pipeline logic is only half of the job; the **storage layer** decides whether results are correct, queryable and fast, and the **ML layer** decides whether the data turns into decisions. This area groups the two "what comes after ETL" topics:

1. **Reliable data lakes.** Plain files on object storage are cheap and open but have no transactions: a failed job leaves half-written files, concurrent writers corrupt each other, schemas drift, and there is no way to update or delete a row. Databases solve this but do not scale to the volume or variety of raw data. The **lakehouse** idea combines both: open files (Parquet) plus a transaction log that adds ACID guarantees, schema enforcement/evolution, and time travel.
2. **Machine learning with MLlib.** Spark's `spark.ml` package provides distributed feature engineering, estimators and pipelines on top of DataFrames, so models can be trained on data too large for a single machine and the data preparation code is shared with the ETL.

Both build on the earlier nodes: DataFrames (structured APIs), the execution model, caching for iterative training, and Structured Streaming for continuously updated tables and online features. The child node *MLlib Pipelines* goes deep on the ML API.

### 2. Core concepts and vocabulary
- **Database vs data lake** — a database offers schema-on-write, ACID and fast SQL on structured data but scales and handles unstructured data poorly; a lake offers cheap, scalable files with schema-on-read but weak reliability.
- **Lakehouse** — lake storage plus a table format with transactions and metadata, queried by SQL engines and Spark.
- **Table format / transaction log** — an ordered record of commits (add file, remove file, change schema) that defines the current table state (Delta Lake, Apache Iceberg, Apache Hudi).
- **ACID** — atomic, consistent, isolated, durable writes; on lakes implemented with optimistic concurrency over the log.
- **Schema enforcement vs evolution** — reject mismatching writes by default; opt in to adding columns (`mergeSchema`).
- **Time travel** — query an earlier table version by version number or timestamp.
- **Upsert / MERGE** — update-or-insert rows by key; makes CDC and slowly changing dimensions practical on a lake.
- **MLlib / `spark.ml`** — the DataFrame-based ML library (the RDD-based `spark.mllib` is in maintenance mode).
- **Transformer / Estimator / Pipeline** — transformers map DataFrames to DataFrames; estimators `fit()` data and return a transformer (model); a pipeline chains stages.
- **Feature vector** — the single `Vector` column of numeric features that most MLlib algorithms expect (built with `VectorAssembler`).
- **Hyperparameter tuning** — grid search plus cross-validation, parallelisable across models.
- **Pandas UDF** — vectorised Python function executed on Arrow batches; used to apply single-node models at scale for batch inference.

### 3. How it works, step by step
**A. Reliable table write (lakehouse)**
1. A job writes new Parquet data files to the table directory — invisible to readers so far.
2. It then attempts to commit one atomic entry to the transaction log listing the files added/removed.
3. If another writer committed first, optimistic concurrency checks for conflicts and retries or fails; readers always see a complete, consistent snapshot.
4. Readers resolve the table by reading the log (plus periodic checkpoints), then scan only the listed files, using per-file statistics to skip data.
5. Maintenance (compaction, vacuum of unreferenced files) runs separately.

**B. Training and serving a model**
1. Clean and join data with DataFrames; split into train/test.
2. Build feature stages (index categoricals, one-hot encode, assemble a vector, scale).
3. Add an estimator (linear model, tree, random forest, gradient boosting…) as the last stage; wrap everything in a `Pipeline`.
4. `fit()` on the training set produces a `PipelineModel`; evaluate on the test set; tune with `CrossValidator`.
5. Save the model (`write().save`), load for batch scoring (`transform`) or streaming scoring, or export for external serving.

### 4. Worked example
```python
# --- Lakehouse: upsert change records into a Delta table (PySpark) ---
from delta.tables import DeltaTable

updates = spark.read.json("/landing/customers_changes")
target  = DeltaTable.forPath(spark, "/lake/silver/customers")

(target.alias("t")
   .merge(updates.alias("s"), "t.id = s.id")
   .whenMatchedUpdateAll()
   .whenNotMatchedInsertAll()
   .execute())

spark.sql("DESCRIBE HISTORY delta.`/lake/silver/customers`").show()   # commits
spark.read.format("delta").option("versionAsOf", 3).load("/lake/silver/customers")  # time travel
```
```python
# --- MLlib pipeline: price prediction with a random forest ---
from pyspark.ml import Pipeline
from pyspark.ml.feature import StringIndexer, VectorAssembler
from pyspark.ml.regression import RandomForestRegressor
from pyspark.ml.evaluation import RegressionEvaluator

train, test = df.randomSplit([0.8, 0.2], seed=42)
idx = StringIndexer(inputCol="room_type", outputCol="room_idx", handleInvalid="skip")
vec = VectorAssembler(inputCols=["bedrooms", "bathrooms", "room_idx"], outputCol="features")
rf  = RandomForestRegressor(labelCol="price", numTrees=50, maxDepth=8, seed=42)

model = Pipeline(stages=[idx, vec, rf]).fit(train)
pred  = model.transform(test)
print(RegressionEvaluator(labelCol="price", metricName="rmse").evaluate(pred))
print(model.stages[-1].featureImportances)
```
How to read it: the merge commits one atomic log entry; `DESCRIBE HISTORY` shows operations and versions. In the ML example the *same* `model` object applies the indexer, assembler and forest to new data, preventing train/serve skew. For a single decision tree, `toDebugString` prints the if/else rules, with numeric splits as `<=` thresholds and categorical splits as set membership.

### 5. Under the hood
- **Why object-store reliability is hard:** listing and renaming on S3-style stores are not atomic, so classic "write to temp, rename" commit protocols are unsafe. Table formats avoid rename by committing a metadata entry instead.
- **Delta/Iceberg/Hudi** all track file-level metadata and statistics (min/max per column) enabling data skipping, and support compaction (`OPTIMIZE`) and clustering/Z-ordering to colocate related rows.
- **Schema enforcement** checks the write against the table schema in the log; evolution is a logged metadata change.
- **Random forest internals:** each tree trains on a bootstrap row sample (bagging, `subsamplingRate`) and considers only a random subset of features at each split (≈ √n for classification, ≈ n/3 for regression), decorrelating trees; predictions are averaged or voted. MLlib trains trees by distributing the *binning* of features (`maxBins`), aggregating histogram statistics per node level across the cluster, which is why `maxBins` and `maxDepth` dominate cost and memory.
- **Distributed ML limits:** iterative algorithms re-scan the data every iteration (cache the training set); linear models parallelise gradient aggregation across partitions; results can differ slightly from scikit-learn because of different implementations and defaults.
- **Pandas UDFs / `applyInPandas`** let you train or score many small single-node models in parallel — one model per group — using Arrow for zero-copy transfer.

### 6. Performance and design implications
- Compact small files; lakes made of millions of tiny files are slow because of listing and task overhead.
- Partition tables on low-cardinality columns that queries filter by (date); avoid over-partitioning.
- Cache the training DataFrame; keep `maxBins`/`maxDepth`/`numTrees` modest and tune with cross-validation using `parallelism` to run model fits concurrently.
- Prefer `VectorAssembler` + built-in estimators over per-row Python; use Pandas UDFs (not row UDFs) for external models.
- Keep feature logic in a persisted pipeline to guarantee identical transformations at training and serving time.
- Track experiments and model versions (MLflow or equivalent) so a model can be reproduced against a given table version.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `df.write.format("delta").save(path)` | Create/append an ACID table | Reliable lake tables |
| `MERGE INTO … USING … ON …` / `DeltaTable.merge` | Upsert/delete by key | CDC, SCD |
| `.option("mergeSchema","true")` | Allow schema evolution on write | Additive column changes |
| `versionAsOf` / `timestampAsOf` | Time-travel read | Audit, rollback, reproducibility |
| `OPTIMIZE`, `VACUUM` | Compact small files / delete old files | Table maintenance |
| `Pipeline`, `VectorAssembler`, `StringIndexer` | Build feature + model pipelines | Any MLlib workflow |
| `CrossValidator(parallelism=n)` | Tune hyperparameters | Model selection |
| `model.write().overwrite().save(p)` / `PipelineModel.load(p)` | Persist model | Batch/streaming scoring |
| `spark.sql.execution.arrow.pyspark.enabled` | Arrow transfer for Python | Pandas UDFs, `toPandas` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Lakehouse table format for shared, updated, audited data | A write-once archive that is only scanned: plain Parquet suffices |
| Delta/Iceberg/Hudi (pick by engine ecosystem) | Mixing multiple writers/engines without checking format support |
| MLlib for large data and shared ETL code | Small data: scikit-learn on one node is faster and richer |
| Distributed training with tree/linear models | Deep learning: use dedicated frameworks and use Spark for data prep |
| Pandas UDF for per-group models | A single model that needs the entire dataset in memory |

### 9. Common mistakes
- Writing raw Parquet with concurrent jobs → partial/overlapping files → use a transactional table format.
- Leaving small files unattended → slow scans → schedule compaction.
- Feature code duplicated in training and serving → skew → serialise the whole `PipelineModel`.
- Fitting encoders/scalers on the full dataset before splitting → leakage → fit inside the pipeline on training data only.
- Not caching the training set → each iteration re-reads and recomputes → cache it.
- Forgetting `handleInvalid` on indexers → job fails on unseen categories at scoring time.
- Comparing metrics with scikit-learn without aligning parameters → misleading differences.

### 10. 🎯 Interview kit
**30-second answer:** A lakehouse puts a transaction log over Parquet files on object storage to give a data lake ACID writes, schema enforcement, time travel and upserts; Spark reads and writes it like any table. MLlib adds distributed feature engineering and model training through DataFrame-based pipelines, so one platform covers raw data to model.
**Structure a longer answer:** clarify the use case (analytics, CDC, ML) → limits of plain lake and of a database → log-based table mechanics → pipeline design for ML with train/serve consistency → maintenance and trade-offs → scenario.
**Likely follow-ups:**
- *How does Delta get atomic commits on S3?* → data files first, then an atomic log entry with optimistic concurrency, not rename.
- *Schema enforcement vs evolution?* → reject mismatches by default; evolve explicitly.
- *Estimator vs transformer?* → estimator `fit()`s and returns a transformer.
- *Why does a random forest generalise better than one tree?* → bagging plus random feature subsets decorrelate trees; averaging reduces variance.
- *How would you apply a scikit-learn model to a billion rows?* → Pandas UDF/`mapInPandas` on Arrow batches.
**Signals interviewers look for:** knowing why plain lakes fail, being able to explain log-based ACID, awareness of leakage and train/serve skew, and sensible judgement about when Spark ML is the wrong tool.

### 11. Check yourself
- [ ] List three failures of a plain Parquet lake that a transaction log removes.
- [ ] Walk through a concurrent write conflict in an optimistic-concurrency table.
- [ ] Build a three-stage MLlib pipeline and name which stage is the estimator.
- [ ] Explain the two randomisations in a random forest and what each decorrelates.
- [ ] Decide between MLlib, scikit-learn and a Pandas UDF for three given data sizes.
<!-- explainer:end -->

## In this section (study in order)

1. [Reliable Data Lakes with Spark](<Reliable_Data_Lakes/_index.md>) (L4 Advanced)
2. [MLlib Pipelines](<MLlib_Pipelines/_index.md>) (L4 Advanced)

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Building Reliable Data Lakes with Apache Spark › Chapter 9. Building Reliable Data Lakes with Apache Spark
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Spark MLlib
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Chapter 10. Machine Learning with MLlib
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › Popular Spark use cases
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — About the Authors
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — E
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Tree-Based Models
- *Fundamentals of Data Engineering* — Joe Reis, Matt Housley — The Data Engineering Lifecycle › Machine learning
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › Pandas UDFs
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Introduction to Apache Spark A Unified Analytics Engine › In Python

