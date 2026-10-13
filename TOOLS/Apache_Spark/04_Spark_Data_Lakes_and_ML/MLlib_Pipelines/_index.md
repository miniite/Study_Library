---
id: "TOOLS/Apache_Spark/04_Spark_Data_Lakes_and_ML/MLlib_Pipelines"
level: 4
type: "curriculum"
curriculum: "data_engineering"
---

# MLlib Pipelines

**L4 Advanced** · [↑ Spark Data Lakes and Machine Learning](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** An MLlib pipeline chains feature transformers and an estimator into one object that is fitted once on training data and then reused identically for evaluation, batch scoring, streaming scoring and deployment.

### 1. The big picture
Real ML work is mostly data preparation: encoding categories, assembling numeric columns into feature vectors, scaling, then fitting a model and checking it. If those steps are scattered in notebooks, the preparation done at training time differs from what happens at serving time (*train/serve skew*), and results cannot be reproduced. MLlib's `pyspark.ml` API solves this with the **Pipeline**: a sequence of stages operating on DataFrames, serialisable as a unit. The pipeline is also where distribution matters: transformers apply in parallel to partitions; estimators aggregate statistics or gradients across the cluster.

Chapter-level themes: designing pipelines, creating them (indexing, one-hot encoding, assembling), evaluating and tuning, then managing and deploying models (MLflow, batch, streaming, model-export, Pandas UDFs).

### 2. Core concepts and vocabulary
- **Transformer** — has `transform(df)`; maps a DataFrame to a DataFrame (e.g. `VectorAssembler`, a fitted model).
- **Estimator** — has `fit(df)`; learns from data and returns a Transformer (e.g. `StringIndexer`, `LinearRegression`).
- **Pipeline** — an Estimator composed of ordered stages; `fit` returns a **PipelineModel** (a Transformer).
- **Param / ParamMap** — hyperparameters set on stages; `ParamGridBuilder` builds search grids.
- **Evaluator** — computes a metric from predictions (RMSE, R², AUC, accuracy, F1).
- **Feature vector** — one `Vector` column (dense or sparse) named `features` by convention.
- **One-hot encoding** — turn category index into a sparse binary vector.
- **CrossValidator / TrainValidationSplit** — hyperparameter tuning with k-fold or single split.
- **MLflow** — open-source tracking/registry for parameters, metrics, models and artifacts.
- **Pandas UDF** — vectorised Python UDF using Arrow, used for scoring single-node models in parallel.

### 3. How it works, step by step
1. **Split** the data into training and test sets (`randomSplit` with a seed; or time-based splits for temporal data).
2. **Define feature stages:**
   - `StringIndexer` → numeric index for each category (`handleInvalid="skip"|"keep"|"error"`),
   - `OneHotEncoder` → sparse vector (needed for linear models; tree models can use indices directly),
   - `VectorAssembler` → combine selected numeric columns and encoded vectors into `features`,
   - optional `StandardScaler`, imputation, `Bucketizer`, text tokenisers/TF-IDF.
3. **Add an estimator** such as `LinearRegression`, `LogisticRegression`, `DecisionTree*`, `RandomForest*`, `GBTClassifier`.
4. **Fit:** `Pipeline(stages=[...]).fit(train)` fits each estimator in order (earlier transformers are applied to data before later stages are fitted) → `PipelineModel`.
5. **Transform / evaluate:** `model.transform(test)` appends `prediction` (and `probability`) columns; an evaluator scores them.
6. **Tune:** wrap the pipeline in `CrossValidator(estimator, estimatorParamMaps, evaluator, numFolds, parallelism)`.
7. **Persist and deploy:** `model.write().overwrite().save(path)`; load with `PipelineModel.load`; use for batch, streaming or export to MLeap/ONNX or serve via MLflow.

### 4. Worked example
```python
from pyspark.ml import Pipeline
from pyspark.ml.feature import StringIndexer, OneHotEncoder, VectorAssembler
from pyspark.ml.regression import RandomForestRegressor
from pyspark.ml.evaluation import RegressionEvaluator
from pyspark.ml.tuning import CrossValidator, ParamGridBuilder
import mlflow, mlflow.spark

train, test = listings.randomSplit([0.8, 0.2], seed=42)

cat_cols = ["room_type", "neighbourhood"]
num_cols = ["bedrooms", "bathrooms", "number_of_reviews"]

indexers = [StringIndexer(inputCol=c, outputCol=c + "_idx", handleInvalid="skip") for c in cat_cols]
encoder  = OneHotEncoder(inputCols=[c + "_idx" for c in cat_cols],
                         outputCols=[c + "_ohe" for c in cat_cols])
assembler = VectorAssembler(inputCols=[c + "_ohe" for c in cat_cols] + num_cols, outputCol="features")
rf = RandomForestRegressor(labelCol="price", featuresCol="features", maxBins=40, seed=42)

pipe = Pipeline(stages=indexers + [encoder, assembler, rf])
grid = (ParamGridBuilder().addGrid(rf.maxDepth, [4, 8]).addGrid(rf.numTrees, [20, 60]).build())
cv = CrossValidator(estimator=pipe, estimatorParamMaps=grid, numFolds=3, parallelism=4,
                    evaluator=RegressionEvaluator(labelCol="price", metricName="rmse"))

with mlflow.start_run(run_name="rf_price"):
    cvModel = cv.fit(train)
    rmse = RegressionEvaluator(labelCol="price", metricName="rmse").evaluate(cvModel.transform(test))
    mlflow.log_metric("rmse", rmse)
    mlflow.spark.log_model(cvModel.bestModel, "model")

scored = cvModel.bestModel.transform(new_data)          # batch scoring
```
Interpretation: `cvModel.bestModel.stages[-1].featureImportances` aligns with the assembler's input order; `avgMetrics` shows each grid point's RMSE. Setting `handleInvalid="skip"` drops unseen categories at scoring (use `"keep"` to map to an extra index). The `maxBins` value must be at least the largest category count for tree models, or training errors.

### 5. Under the hood
- **How stages execute:** `fit` walks stages in order; for a Transformer stage it simply transforms the training DataFrame; for an Estimator it calls `fit` on the transformed data and replaces it with the fitted model. The `PipelineModel` holds only fitted transformers.
- **Distributed algorithms:** linear/logistic regression use L-BFGS/OWL-QN, aggregating gradients over partitions with `treeAggregate`; tree ensembles bin continuous features (`maxBins`) once, then each level of the tree is built by aggregating per-node histograms across the cluster; ALS uses blocked matrix factorisation. Iterative algorithms re-scan the data each iteration → cache training data (`train.cache()`).
- **Sparse vs dense vectors:** one-hot output is sparse; `VectorAssembler` preserves sparsity; memory and speed depend on it.
- **Tree vs scikit-learn differences:** MLlib trees use approximate binned splits and limits on depth (≤ 30), so results differ even with matching hyperparameters.
- **Model persistence format:** metadata JSON + Parquet data per stage; portable between Spark versions only within compatibility limits, so record the Spark version.
- **Deployment options:** (1) batch scoring on a schedule; (2) streaming: `model.transform(streamingDF)` inside a Structured Streaming query (stateless transformer); (3) low-latency online serving: export the model (MLeap, ONNX, PMML) or train a lighter model outside Spark, because Spark's `transform` has job-scheduling latency; (4) managed serving through MLflow flavours (`mlflow.spark`, `pyfunc`).
- **Pandas UDF / `applyInPandas` patterns** for single-node libraries: broadcast a model or load per partition, score Arrow batches; or train one model per group in parallel.

### 6. Performance and design implications
- Cache the training DataFrame; avoid recomputing expensive feature logic each iteration.
- Prefer built-in transformers over Python UDFs; use Arrow-based Pandas UDFs where unavoidable.
- Use `CrossValidator(parallelism=n)` to fit candidate models concurrently, and `TrainValidationSplit` to save time on large data.
- Reduce dimensionality of one-hot features for high-cardinality columns (hashing, target encoding, frequency thresholds).
- Keep identical feature code for training and scoring by always shipping the `PipelineModel`.
- Check class imbalance, leakage (fit encoders only on training data inside the pipeline) and set seeds for reproducibility.
- Track data version (table version/time travel), code version, parameters, metrics and model artefacts together (MLflow).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `Pipeline(stages=[...]).fit(df)` | Fit all stages | Always package preprocessing + model |
| `StringIndexer`, `OneHotEncoder`, `VectorAssembler`, `StandardScaler` | Feature engineering | Prepare input vectors |
| `CrossValidator`, `ParamGridBuilder`, `TrainValidationSplit` | Tuning | Hyperparameter search |
| `RegressionEvaluator`, `BinaryClassificationEvaluator`, `MulticlassClassificationEvaluator` | Metrics | Model comparison |
| `model.write().overwrite().save(p)` / `PipelineModel.load(p)` | Persist/load | Reuse and deployment |
| `model.stages[-1].toDebugString`, `.featureImportances` | Inspect trees/importances | Interpretation |
| `mlflow.start_run()`, `mlflow.spark.log_model` | Tracking and registry | Reproducibility |
| `pandas_udf`, `mapInPandas`, `applyInPandas` | Scale single-node models | Batch inference |
| `df.cache()` | Hold training data | Iterative algorithms |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| MLlib pipelines for large data and Spark-native ETL | Small data (< a few GB): scikit-learn on one machine is simpler and often more accurate/faster |
| Tree ensembles on mixed features | Very high-dimensional sparse text with linear models might be faster with hashing |
| Streaming scoring with `transform` | Single-request low-latency serving → export or serve outside Spark |
| Pandas UDF for pretrained models | Models that don't fit executor memory |
| CrossValidator on small grids | Large grids × folds on big data are costly → random search/subsampling |

### 9. Common mistakes
- Fitting preprocessing on the whole dataset before splitting → leakage → keep it inside the pipeline and fit on train only.
- Forgetting `handleInvalid` → scoring job fails on new categories → choose `skip` or `keep`.
- `maxBins` lower than category cardinality → training error → raise it or reduce cardinality.
- One-hot encoding high-cardinality fields → huge sparse vectors → hashing or grouping.
- Not caching training data → each iteration recomputes → cache.
- Saving only the model without the feature stages → can't reproduce features at serving → save the `PipelineModel`.
- Comparing metrics with a different library without matching parameters → misleading.

### 10. 🎯 Interview kit
**30-second answer:** An MLlib pipeline is a chain of stages — transformers like indexers and assemblers and a final estimator — that is fitted once into a PipelineModel, so exactly the same preprocessing and model run in evaluation, batch and streaming scoring. Tuning uses CrossValidator with a parameter grid, and models are tracked and deployed with MLflow or exported for low-latency serving.
**Structure a longer answer:** clarify data size and serving needs → Pipeline/Estimator/Transformer model → feature stages and model choice → evaluation and tuning → deployment options and trade-offs → pitfalls (leakage, skew).
**Likely follow-ups:**
- *Estimator vs transformer?* → fit produces a transformer.
- *How do you avoid train/serve skew?* → persist and reuse the PipelineModel.
- *How do you deploy for real-time?* → export (ONNX/MLeap) or serve outside Spark; Spark for batch/streaming.
- *Why can MLlib differ from scikit-learn?* → different algorithms/binning/defaults.
- *How to scale single-node models?* → Pandas UDFs, per-group training with `applyInPandas`.
**Signals interviewers look for:** clear object model, leakage awareness, realistic deployment thinking, and knowing when not to use Spark ML.

### 11. Check yourself
- [ ] Build a pipeline with an indexer, encoder, assembler and a tree regressor and read the feature importances.
- [ ] Explain what `Pipeline.fit` does for each stage type.
- [ ] Name where leakage can occur and how pipelines prevent it.
- [ ] Choose a deployment option for batch nightly scoring vs 20 ms API latency.
- [ ] Set up `CrossValidator` with a two-parameter grid and compute the number of models fitted.
<!-- explainer:end -->

## References

- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › Chapter 11. Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › Model Deployment Options with MLlib
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Designing Machine Learning Pipelines
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — M
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › Batch
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Creating a Pipeline
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › In Python
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Managing, Deploying, and Scaling Machine Learning Pipelines with Apache Spark › MLflow
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › One-hot encoding
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Machine Learning with MLlib › Using Estimators to Build Models

