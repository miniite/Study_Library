---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd promote a single validated Spark/dbt artifact through dev, staging, and prod warehouses without recompiling, and what environment-specific config you'd inject at run time.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Build Reproducibility and Artifact Promotion](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Build_and_Release_Metadata/Build_Reproducibility_and_Artifact_Promotion.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To promote a validated Spark or dbt artifact through development (dev), staging, and production environments without recompiling, we need to ensure that the pipeline is fully decoupled from environment-specific configurations. This involves using environment variables and parameterization in our code, along with robust CI/CD pipelines to automate deployment. We'll inject specific configurations at runtime based on which environment the artifact is being deployed to.

This approach ensures that there's no recompilation required across environments and maintains a consistent and reliable workflow. The key steps are:

Parameterize all configuration options.
Use environment variables for dynamic settings.
Implement CI/CD pipelines with version control and automated testing.
Deploy the artifact in each environment, injecting specific configurations.

**Comparison:**

```
Environment	Source Path	Target Path
Dev	s3a://data-lake/dev/source/data.csv	s3a://data-lake/dev/transformed
Staging
Prod
```

**Code:**

```python
# Example of parameterizing Spark job
from pyspark.sql import SparkSession

def main(spark, config):
    # Load data from source
    df = spark.read.format(config['format']).options(**config['options']).load(config['path'])

    # Transform and save to target
    transformed_df = transform(df)
    transformed_df.write.mode('overwrite').save(config['target_path'])

# Example of environment-specific configuration
dev_config = {
    "format": "csv",
    "options": {"header": True, "inferSchema": True},
    "path": "s3a://data-lake/dev/source/data.csv",
    "target_path": "s3a://data-lake/dev/transformed"
}

staging_config = {
    # Different configurations for staging environment
}
prod_config = {
    # Production-specific configurations
}
```

**Example input / output:**

```text
# Example CI/CD pipeline steps

# Step 1: Build and test the artifact in dev environment
git checkout master
make build
make test

# Step 2: Deploy to staging environment
export SPARK_ENV_VARIABLES=$(cat ./config/staging.json)
spark-submit --conf "spark.hadoop.fs.s3a.access.key=$ACCESS_KEY" --conf "spark.hadoop.fs.s3a.secret.key=$SECRET_KEY" /path/to/artifact.py $SPARK_ENV_VARIABLES

# Step 3: Manual testing and validation in staging
make validate_staging

# Step 4: Promote to prod after successful testing in staging
export SPARK_ENV_VARIABLES=$(cat ./config/prod.json)
spark-submit --conf "spark.hadoop.fs.s3a.access.key=$ACCESS_KEY" --conf "spark.hadoop.fs.s3a.secret.key=$SECRET_KEY" /path/to/artifact.py $SPARK_ENV_VARIABLES

# Step 5: Monitor and validate in production
make validate_prod
```

**Data:**

```json
{
  "dev_config": {
    "format": "csv",
    "options": {"header": true, "inferSchema": true},
    "path": "s3a://data-lake/dev/source/data.csv",
    "target_path": "s3a://data-lake/dev/transformed"
  },
  "staging_config": {
    // Different configurations for staging environment
  },
  "prod_config": {
    // Production-specific configurations
  }
}
```

**Flow:**

1. CI/CD Pipeline
2. Dev Build and Test
3. Deploy to Dev
4. Monitor Dev
5. Staging Deployment
6. Monitor Staging
7. Promote to Prod
8. Monitor Prod

