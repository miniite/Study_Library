---
type: "practice"
difficulty: "Deep"
---

# boto3 throttles you with ClientError ThrottlingException under a large batch operation. How would you implement adaptive retry and back off correctly, and what does botocore's built-in retry config already give you?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Rate-limited Integrations](<../../../../TOOLS/Python/Python_Language/Rate-limited_Integrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with AWS services via boto3, encountering ThrottlingException is a common issue, especially during large-scale batch operations. The goal here is to implement adaptive retry and back-off strategies that respect AWS rate limits while ensuring efficient execution of the operation.

Firstly, let's understand what botocore’s built-in retry configuration offers and where it falls short in our context. Then we'll explore how to craft a robust solution using these insights.

#### Triage
What the Question Is Asking

The question is probing your understanding of boto3's handling of rate limiting, specifically through the ThrottlingException. It also asks you to design an adaptive retry mechanism that intelligently back off and retry operations based on AWS’s response. Additionally, it aims to assess how well you understand botocore's built-in retry configuration.

#### Assumptions
You are using a large batch operation where multiple requests are made.
The operation involves frequent ThrottlingException errors.
You need to handle retries and back-off in an efficient manner without overwhelming AWS services or your application.

#### First-Principles Solution
Step 1: Understanding the Built-In Retry Mechanism

Botocore, the low-level client for Amazon Web Services (AWS), comes with a built-in retry mechanism that can automatically handle some rate limiting issues. By default, it retries operations up to five times if certain exceptions are raised, including ThrottlingException. However, this simple exponential back-off strategy might not be sufficient for large batch operations.

Step 2: Custom Retry Logic

To implement adaptive retry and back-off correctly:

Retry Strategy: Implement a custom retry mechanism that can intelligently adjust the number of retries based on the error rate.
Back-Off Strategy: Use an exponential back-off algorithm to increase the waiting time between retries, but not indefinitely.

Here’s how you could structure this in Python:

PYTHON
import time
from botocore.config import Config
from botocore.exceptions import ClientError

def adaptive_retry(func):
    def wrapper(*args, **kwargs):
        max_retries = 10  # Define the maximum number of retries
        retry_backoff_factor = 2  # Initial back-off factor
        retry_count = 0

        while True:
            try:
                return func(*args, **kwargs)
            except ClientError as e:
                if "ThrottlingException" in str(e):
                    if retry_count >= max_retries:
                        raise Exception("Max retries exceeded.")

                    # Update back-off factor based on error rate
                    if retry_count > 0 and retry_count % 5 == 0:  # Increase back-off after every 5 retries
                        retry_backoff_factor *= 2

                    time.sleep(retry_backoff_factor)
                    retry_count += 1
                else:
                    raise e

    return wrapper

@adaptive_retry
def batch_operation(s3_client, bucket_name, keys):
    for key in keys:
        s3_client.download_file(bucket_name, key, f'/tmp/{key}')
Step 3: Reasoning Through First Principles
Error Handling: The ClientError is caught to specifically handle ThrottlingException.
Retry Logic: Retrying the operation until a maximum number of retries (max_retries) is reached.
Back-off Mechanism: Exponential back-off with an initial factor and dynamic adjustment based on retry count.
Bucketing Errors: After every 5 retries, the back-off factor is doubled to ensure more significant delays.

#### Closing Summary

This custom retry mechanism ensures that your application can handle ThrottlingException effectively by implementing adaptive retries and a growing back-off strategy. It leverages botocore's built-in retry configuration while enhancing it with smarter handling of rate limiting errors.

#### Common Mistakes/Follow-Ups
Infinite Loop: Ensuring the maximum number of retries is set to avoid infinite loops.
Back-off Factor: Dynamically adjusting the back-off factor based on error patterns can lead to better performance and less resource contention.
Rate Limit Awareness: Understanding AWS service rate limits and designing your retry logic accordingly.

**Code:**

```python
import time

def adaptive_retry(func):
    def wrapper(*args, **kwargs):
        max_retries = 10
        retry_backoff_factor = 2
        retry_count = 0

        while True:
            try:
                return func(*args, **kwargs)
            except ClientError as e:
                if "ThrottlingException" in str(e):
                    if retry_count >= max_retries:
                        raise Exception("Max retries exceeded.")

                    if retry_count > 0 and retry_count % 5 == 0:
                        retry_backoff_factor *= 2

                    time.sleep(retry_backoff_factor)
                    retry_count += 1
                else:
                    raise e

    return wrapper

@adaptive_retry
def batch_operation(s3_client, bucket_name, keys):
    for key in keys:
        s3_client.download_file(bucket_name, key, f'/tmp/{key}')
```

**Example input / output:**

```text
# Example input and output
keys = ['file1.txt', 'file2.jpg', 'large_file.zip']
s3_client = boto3.client('s3')  # Assume properly configured

batch_operation(s3_client, 'my-bucket', keys)

# Expected response (assuming no errors)
# Files are downloaded from S3 to the local /tmp directory
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is throttling exception?
7. Retry with increased back-off
8. Maximum retries exceeded?
9. Handle error or notify user
10. Operation successful

