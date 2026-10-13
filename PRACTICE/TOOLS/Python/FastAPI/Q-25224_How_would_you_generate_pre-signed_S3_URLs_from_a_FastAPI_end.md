---
type: "practice"
difficulty: "Deep"
---

# How would you generate pre-signed S3 URLs from a FastAPI endpoint so clients can upload directly to storage?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ API Authentication](<../../../../TOOLS/Python/FastAPI/API_Authentication.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To generate pre-signed S3 URLs from a FastAPI endpoint, we need to follow several steps that involve both security and functionality. This task requires understanding of AWS SDK integration with FastAPI, and handling credentials securely. Here's how you can approach it:

Firstly, let’s summarize the core idea: We'll create an API in FastAPI that generates pre-signed URLs for S3 buckets, allowing clients to upload files directly without needing direct access to S3.

Now, let’s dive into the details of triaging the question and the solution step-by-step.

**Comparison:**

| Parameter | Description |
|-----------|-------------|
| Bucket    | S3 bucket name. |
| Key       | The key or path where the object will be stored in the bucket. |
| ExpiresIn | Expiration time for the URL, in seconds. Default is 1800 (30 minutes). Can be adjusted up to a maximum of one day. |

**Code:**

```python
from fastapi import FastAPI, HTTPException
import boto3
from botocore.config import Config
from pydantic import BaseModel

app = FastAPI()

class S3UploadRequest(BaseModel):
    file_name: str
    bucket_name: str
```

**Example input / output:**

```text
@app.post("/pre-signed-url/")
def generate_presigned_url(request_data: S3UploadRequest):
    # Configuration for the AWS client
    s3_client = boto3.client(
        's3',
        aws_access_key_id="YOUR_ACCESS_KEY",
        aws_secret_access_key="YOUR_SECRET_KEY",
        config=Config(signature_version='s3v4')
    )

    try:
        # Generate pre-signed URL with a 10-minute validity period
        presigned_url = s3_client.generate_presigned_url(
            'put_object',
            Params={'Bucket': request_data.bucket_name, 'Key': request_data.file_name},
            ExpiresIn=600
        )

        return {"presigned_url": presigned_url}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

**Data:**

```
Sample input data:
JSON
Copy
{
    "file_name": "example.txt",
    "bucket_name": "my-bucket"
}
```

**Flow:**

1. User Request
2. S3UploadRequest Model
3. Parsing Request Data
4. S3Client Initialization
5. Generate Presigned URL

