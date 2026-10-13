---
type: "practice"
difficulty: "Intermediate"
---

# Your Python job that snapshots RDS/EBS volumes nightly is silently failing on some accounts. How would you make it observable, alert on partial failure, and ensure one account's failure doesn't skip the rest?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Graceful Shutdown](<../../../../TOOLS/Python/Python_Language/Graceful_Shutdown.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of silent failures in a Python job that snapshots RDS/EBS volumes nightly across multiple AWS accounts, we need to make the job observable, alert on partial failures, and ensure that one account's failure does not cascade to others. This involves leveraging logging, error handling, and parallel execution strategies.

First, let’s summarize what we're tackling: our Python script needs robust observability to identify issues early, reliable alerts for critical problems, and a resilient architecture that ensures no single account's failure affects the entire process.

#### Triaging the Question

The core of this question is about improving the reliability and observability of an existing AWS Lambda function or similar Python-based job. The interviewer is probing your understanding of logging practices, error handling strategies, parallel execution, and alerting mechanisms.

#### Assumptions
We are using AWS services for running our Python script.
Each account has its own set of RDS/EBS volumes to be snapshotted.
The failure could occur due to various reasons like insufficient permissions, volume not found, or network issues.
We aim to log and alert on failures at the individual account level rather than lumping them all together.

#### First-Principles Solution
1. Logging and Monitoring

We will enhance our Python script with comprehensive logging using libraries like logging or boto3's built-in support for logging. This ensures that we capture detailed information about the state of each account and any potential issues.

PYTHON
import boto3
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def snapshot_account_volumes(account_id):
    try:
        # Code to snapshot RDS/EBS volumes in the specified AWS account
        pass
    except Exception as e:
        logger.error(f"Failed to snapshot volumes for account {account_id}: {str(e)}")
2. Error Handling and Retries

Implement error handling to manage transient issues like network disruptions or temporary permissions changes. This can be done using exponential backoff strategies.

PYTHON
from time import sleep

def snapshot_account_volumes(account_id):
    try:
        # Code to snapshot RDS/EBS volumes in the specified AWS account
    except Exception as e:
        logger.error(f"Initial failure: {str(e)}")
        for retry_attempt in range(3):  # Retry up to 3 times
            sleep(2 ** retry_attempt)  # Exponential backoff
            try:
                # Code to snapshot RDS/EBS volumes again
                break
            except Exception as e:
                logger.error(f"Retry {retry_attempt + 1} failed: {str(e)}")
        else:
            raise RuntimeError("Failed after multiple retries")
3. Parallel Execution and Dependency Management

Use AWS Lambda's concurrency model to handle parallel execution of accounts, but ensure that the dependencies between them are managed carefully.

PYTHON
import concurrent.futures

def snapshot_accounts(accounts):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        futures = [executor.submit(snapshot_account_volumes, account_id) for account_id in accounts]
        for future in concurrent.futures.as_completed(futures):
            try:
                result = future.result()
            except Exception as e:
                logger.error(f"Error during snapshot: {str(e)}")

snapshot_accounts([account1, account2, account3])
4. Alerting and Monitoring

Set up alerting using AWS CloudWatch or other monitoring tools to trigger notifications when issues arise.

PYTHON
import boto3

def send_alert(account_id):
    cloudwatch = boto3.client('cloudwatch')
    cloudwatch.put_metric_data(
        MetricData=[
            {
                'MetricName': 'SnapshotFailure',
                'Dimensions': [
                    {'Name': 'AccountId', 'Value': account_id},
                ],
                'Unit': 'None',
                'Value': 1
            },
        ],
        Namespace='Custom'
    )

def snapshot_account_volumes(account_id):
    try:
        # Code to snapshot RDS/EBS volumes in the specified AWS account
    except Exception as e:
        logger.error(f"Failed to snapshot volumes for account {account_id}: {str(e)}")
        send_alert(account_id)

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing comprehensive logging, robust error handling with retries, parallel execution management, and alerting mechanisms, we can significantly improve the reliability and observability of our Python job. Key areas to focus on include ensuring that each account's failure is logged distinctly, managing retries effectively, and setting up reliable alerting.

#### Common mistakes
include:

Overlooking transient errors leading to unnecessary retries.
Failing to capture detailed logs for troubleshooting.
Not properly handling dependencies between accounts.
Ignoring alerting mechanisms which can lead to silent failures going unnoticed.

#### Follow-ups
could involve discussing specific AWS services used (e.g., Lambda, CloudWatch), best practices for error handling in multi-threaded environments, and strategies for scaling the job to handle more accounts efficiently.

**Comparison:**

```
Account	Action
account1	Success
account2	Failure (UnauthorizedOperation)
account3	Success
```

**Code:**

```python
import boto3
import logging

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def snapshot_account_volumes(account_id):
    try:
        # Code to snapshot RDS/EBS volumes in the specified AWS account
        pass
    except Exception as e:
        logger.error(f"Failed to snapshot volumes for account {account_id}: {str(e)}")

def send_alert(account_id):
    cloudwatch = boto3.client('cloudwatch')
    cloudwatch.put_metric_data(
        MetricData=[
            {
                'MetricName': 'SnapshotFailure',
                'Dimensions': [
                    {'Name': 'AccountId', 'Value': account_id},
                ],
                'Unit': 'None',
                'Value': 1
            },
        ],
        Namespace='Custom'
    )

def snapshot_accounts(accounts):
    with concurrent.futures.ThreadPoolExecutor(max_workers=5) as executor:
        futures = [executor.submit(snapshot_account_volumes, account_id) for account_id in accounts]
        for future in concurrent.futures.as_completed(futures):
            try:
                result = future.result()
            except Exception as e:
                logger.error(f"Error during snapshot: {str(e)}")
                send_alert(future.result().args[0])
```

**Example input / output:**

```text
accounts = ['account1', 'account2', 'account3']

snapshot_accounts(accounts)
# Logs:
# INFO:root:Failed to snapshot volumes for account account2: ConnectionError('an error occurred (UnauthorizedOperation) when calling the DescribeDBInstances operation: User: arn:aws:sts::123456789012:assumed-role/ExampleRole/User is not authorized to perform: rds:DescribeDBInstances on resource: *')
# ERROR:root:Error during snapshot: ConnectionError('an error occurred (UnauthorizedOperation) when calling the DescribeDBInstances operation: User: arn:aws:sts::123456789012:assumed-role/ExampleRole/User is not authorized to perform: rds:DescribeDBInstances on resource: *')
```

**Data:**

```json
{
  "accounts": ["account1", "account2", "account3"]
}
```

**Flow:**

1. Success
2. Failure
3. Success
4. Failure
5. Start
6. Check Account
7. Snapshot Volumes
8. Log Error & Send Alert
9. End
10. Retry (Up to 3 Times)

