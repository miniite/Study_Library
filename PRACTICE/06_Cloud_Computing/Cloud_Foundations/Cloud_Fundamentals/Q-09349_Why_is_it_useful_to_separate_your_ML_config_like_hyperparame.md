---
type: "practice"
difficulty: "Basic"
---

# Why is it useful to separate your ML config like hyperparameters and paths from the code using environment variables in the cloud?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Reproducible Containers and Artifacts](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Reproducible_Containers_and_Artifacts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a cloud-based machine learning (ML) project, separating hyperparameters and file paths into environment variables is crucial for several reasons. This practice enhances flexibility, security, and maintainability of your codebase. Let's break down why this separation is beneficial.

Firstly, environment variables allow you to configure your application without altering the source code. For instance, if you need to change a path or a hyperparameter value, you can do so by modifying the environment variable settings rather than rewriting parts of your code. This is particularly useful in cloud environments where different deployment stages (development, testing, staging, production) may require different configurations.

Secondly, using environment variables enhances security. Sensitive information like API keys, database credentials, and paths to sensitive files can be stored as environment variables instead of being hard-coded into your application. This is a best practice in cloud environments, where services often support secure storage options for such data. Hard-coding these values directly into your code increases the risk of exposure if your source code is ever compromised.

Thirdly, flexibility and maintainability are significantly improved with this approach. Different cloud environments may require different configurations. For example, a local development environment might use a different path than a production environment on AWS or GCP. By storing these paths in environment variables, you can easily switch between environments without changing your code.

To illustrate, imagine you have an ML model that needs to read data from S3 buckets and write results to specific directories. You would define the relevant paths as environment variables, such as AWS_BUCKET_NAME for the S3 bucket name and MODEL_SAVE_PATH for where the trained model should be saved. This way, you can deploy your application on different cloud providers or in different regions without altering any of your code.

In summary, separating hyperparameters and file paths into environment variables is a best practice that enhances flexibility, security, and maintainability in cloud-based ML projects.

**Code:**

```python
# Example of using environment variables in Python (using os module)
import os

AWS_BUCKET_NAME = os.environ.get('AWS_BUCKET_NAME', 'default-bucket')
MODEL_SAVE_PATH = os.environ.get('MODEL_SAVE_PATH', '/local/path/to/model')
```

**Example input / output:**

```text
# Example input environment variables
export AWS_BUCKET_NAME=my-s3-bucket
export MODEL_SAVE_PATH=/path/to/production/models

# Example output behavior
Model loaded from S3 bucket my-s3-bucket and saved to /path/to/production/models.
```

**Flow:**

1. Configure EnvironmentVariables
2. Deploy Application
3. (Development) Use LocalSettings
4. (Testing) Use Test Settings
5. (Staging & Production) UseProd Settings

