---
type: "practice"
difficulty: "Basic"
---

# How would you upload a 50 GB dataset from your laptop into cloud object storage for training?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Object Storage Classes](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Object_Storage_Classes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To upload a 50 GB dataset from your laptop to cloud object storage, follow these steps:

#### Overview
The goal is to efficiently transfer a large file in a way that minimizes costs and maximizes reliability.

#### Triage
This question assesses your understanding of cloud fundamentals, including data transfer methods, cost considerations, and the use of APIs. You need to consider factors like internet speed, cost, and potential data loss during transmission.

#### First Principles Solution

Choose a Suitable Cloud Service Provider: Providers like AWS S3, Google Cloud Storage, or Azure Blob Storage offer robust services for storing large datasets.
Use the Official SDK or API: These tools provide reliable and efficient ways to upload files directly from your machine.
Verify Internet Connection: Ensure you have a stable and fast internet connection. If not, consider using a different location with better connectivity.
Chunking Large Files (Optional): For very large datasets, chunk the file into smaller parts before uploading to handle potential interruptions.

Step-by-Step Solution:

Install SDK: Install the appropriate SDK for your preferred cloud provider.
BASH
pip install boto3  # For AWS S3
Authenticate: Obtain your API credentials and authenticate with the cloud service.
PYTHON
import boto3

s3 = boto3.client('s3', aws_access_key_id='YOUR_ACCESS_KEY',
                 aws_secret_access_key='YOUR_SECRET_KEY')
Upload File:
PYTHON
def upload_file_to_s3(file_path, bucket_name, object_name):
    with open(file_path, 'rb') as file_data:
        s3.upload_fileobj(file_data, bucket_name, object_name)

#### Closing Summary
By following these steps, you can reliably and efficiently transfer a 50 GB dataset to cloud storage using the official SDKs provided by major cloud providers.

#### Common Mistakes/Follow-Ups

Not Using Chunking: For very large files, not chunking the file could result in timeouts or failure.
Ignoring Network Stability: A weak internet connection can lead to failed uploads and potential data loss.
Failing to Check SDK Documentation: Not understanding all features of the SDK might limit your ability to optimize the transfer process.

**Comparison:**

| Step | Action |
|------|--------|
| 1    | Choose cloud provider and install SDK. |
| 2    | Authenticate with the provider.       |
| 3    | Define a function to upload files.    |
| 4    | Upload file using defined function.   |

**Code:**

```python
import boto3

def upload_file_to_s3(file_path, bucket_name, object_name):
    s3 = boto3.client('s3', aws_access_key_id='YOUR_ACCESS_KEY',
                     aws_secret_access_key='YOUR_SECRET_KEY')

    with open(file_path, 'rb') as file_data:
        s3.upload_fileobj(file_data, bucket_name, object_name)
```

**Example input / output:**

```text
# Example Input
file_path = '/path/to/your/dataset.csv'
bucket_name = 'my-data-bucket'
object_name = '50gb-dataset.csv'

# Example Output
upload_file_to_s3(file_path, bucket_name, object_name)
```

**Data:**

```json
{
  "file_path": "/path/to/your/dataset.csv",
  "bucket_name": "my-data-bucket",
  "object_name": "50gb-dataset.csv"
}
```

**Flow:**

1. Install SDK
2. Authenticate
3. Define Function
4. Upload File

