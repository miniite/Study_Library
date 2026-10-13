---
type: "practice"
difficulty: "Deep"
---

# A training job in CI crashed at hour 6 and left partial artifacts in the registry; how do you design the pipeline so half-baked models never get promoted?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pre-merge vs Post-merge Failures](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Pre-merge_vs_Post-merge_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a situation where a training job in Continuous Integration (CI) crashes partway through, leaving only partial artifacts in your model registry, the primary goal is to design a robust CI/CD pipeline that ensures only fully completed and validated models are promoted into production. This involves several key steps: detecting incomplete jobs, managing partial artifacts, and ensuring rigorous validation before promotion.

To achieve this, we need to focus on three main areas:

Detecting Incomplete Jobs: Setting up mechanisms to identify when a job has failed or been interrupted.
Handling Partial Artifacts: Defining strategies for handling incomplete model checkpoints without promoting them.
Validation Before Promotion: Implementing comprehensive validation checks before models are promoted.

Let's break this down step-by-step from first principles:

#### Triage
What the Question is Really Asking and Assumptions

The question asks how to design a CI/CD pipeline such that half-baked models never get promoted. Key assumptions include:

The training job can crash at any stage.
Partial artifacts (e.g., incomplete model checkpoints) are left behind when a job fails.
We need a system that ensures only fully completed and validated models are deployed.
Step-by-Step Solution

Detecting Incomplete Jobs

Use a status tracking mechanism in the CI pipeline to monitor the state of each training job. This could be as simple as checking if the job has finished successfully or if it was interrupted.
Implement hooks in your CI/CD tools (e.g., Jenkins, GitHub Actions) that notify you when a job fails.

Handling Partial Artifacts

Ensure that partial artifacts are not stored in the model registry unless specifically allowed during an ongoing failure scenario.
Configure your artifact storage to either delete or quarantine incomplete checkpoints automatically upon detection of a failed job.

Validation Before Promotion

Implement rigorous validation checks for models before they are promoted. This could include:
Performance metrics (e.g., accuracy, loss) to ensure the model meets performance thresholds.
Compliance with business rules and constraints.
Integration tests to ensure compatibility with existing systems.
Detailed Implementation
Detecting Incomplete Jobs
Use build status APIs in CI tools like Jenkins or GitHub Actions to track job states. For example:
PYTHON
from jenkinsapi.jenkins import Jenkins

j = Jenkins('http://jenkins.example.com', username, password)

# Get the job object
job = j.get_job('my_training_job')

# Check if the last build was successful
last_build = job.get_last_completed_build()
if not last_build.is_success():
    print("Job failed at", last_build.get_timestamp())
Handling Partial Artifacts
Configure storage mechanisms to automatically delete or quarantine incomplete checkpoints. For example, using AWS S3:
PYTHON
import boto3

s3 = boto3.client('s3')

def remove_incomplete_checkpoints(bucket_name):
    response = s3.list_objects(Bucket=bucket_name)
    for obj in response.get('Contents', []):
        if 'incomplete' in obj['Key']:
            s3.delete_object(Bucket=bucket_name, Key=obj['Key'])
Validation Before Promotion
Implement validation scripts that run as part of the CI pipeline. For instance, using a Python script:
PYTHON
import unittest

class TestModelValidation(unittest.TestCase):
    def test_model_accuracy(self):
        # Load and validate model performance
        pass

    def test_compliance(self):
        # Check business rules and constraints
        pass

if __name__ == '__main__':
    unittest.main()

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, to prevent half-baked models from being promoted, we need a CI/CD pipeline that:

Monitors job status.
Manages partial artifacts effectively.
Conducts thorough validation before promotion.

#### Common mistakes
include not properly handling incomplete checkpoints or skipping rigorous validation steps. Follow-up questions might explore specific tools and techniques used for monitoring, artifact management, and validation in practice.

**Code:**

```

from jenkinsapi.jenkins import Jenkins

def check_job_status(job_name): j = Jenkins('http://jenkins.example.com', username, password)

CODE
Copy
job = j.get_job(job_name)
last_build = job.get_last_completed_build()
if not last_build.is_success():
    print("Job failed at", last_build.get_timestamp())
CODE
Copy

```python
import boto3

def remove_incomplete_checkpoints(bucket_name):
    s3 = boto3.client('s3')

    response = s3.list_objects(Bucket=bucket_name)
    for obj in response.get('Contents', []):
        if 'incomplete' in obj['Key']:
            s3.delete_object(Bucket=bucket_name, Key=obj['Key'])
PYTHON
Copy
import unittest

class TestModelValidation(unittest.TestCase):
    def test_model_accuracy(self):
        # Load and validate model performance
        pass

    def test_compliance(self):
        # Check business rules and constraints
        pass

if __name__ == '__main__':
    unittest.main()
```

**Flow:**

1. Yes
2. No
3. Incomplete
4. Complete
5. Pass
6. Fail
7. Start
8. Is Job Successful?
9. Promote Model
10. Check Partial Artifacts
11. Quarantine Artifact
12. Re-run Validation
13. Do Not Promote Model
14. End

