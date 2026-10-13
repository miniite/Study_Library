---
type: "practice"
difficulty: "Deep"
---

# How would you design an endpoint that needs to return millions of rows to a data pipeline efficiently, comparing pagination, streaming NDJSON, and generating a file plus a signed download URL?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Progressive Results](<../../../../07_Distributed_Systems_and_Design/APIs/REST_APIs/Progressive_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient REST API endpoint that handles returning millions of rows to a data pipeline, we need to consider multiple factors including performance, scalability, reliability, and ease of integration with downstream systems. We will compare three approaches: pagination, streaming NDJSON, and generating a file plus a signed download URL.

#### Triage

The question is probing for an understanding of how to handle large datasets efficiently in the context of REST APIs. The key aspects are:

Pagination: Efficiently managing large result sets by dividing them into manageable chunks.
Streaming NDJSON (Newline Delimited JSON): Sending data in a stream format, which can be more efficient for real-time processing.
Generating a File + Signed Download URL: Providing a pre-generated file that downstream systems can download.

#### First-Principles Solution
1. Pagination

#### Overview
Pagination is straightforward but may not be the most efficient approach when dealing with millions of rows. It involves breaking the data into smaller chunks and returning them page by page.

#### Implementation

PYTHON
def get_users_page(page_size, page_number):
    start_index = (page_number - 1) * page_size
    end_index = start_index + page_size
    users = User.objects.all()[start_index:end_index]
    return [user.to_dict() for user in users]

#### Pros

Simple to implement.
Well-understood by many systems.

#### Cons

May result in high memory usage if all chunks are loaded into memory simultaneously.
Can lead to performance bottlenecks when dealing with large page sizes or numbers of pages.
2. Streaming NDJSON

#### Overview
Streaming data via NDJSON allows for continuous processing and can be more efficient, especially when the pipeline is reading the stream in real-time.

#### Implementation

PYTHON
import json
from flask import Response

def generate_users_stream():
    users = User.objects.all()
    for user in users:
        yield f"{json.dumps(user.to_dict())}\n"

#### Pros

Reduces memory footprint by processing data on-the-fly.
Allows real-time processing and can be more efficient.

#### Cons

Requires the downstream system to handle streaming.
May introduce complexity if not all systems are designed for stream consumption.
3. Generating a File + Signed Download URL

#### Overview
Generating a file with millions of rows and providing a signed download URL is useful when the pipeline needs to process the data at some point in time, rather than continuously.

#### Implementation

PYTHON
import boto3

def create_user_file(bucket_name, file_key):
    s3 = boto3.client('s3')
    users = User.objects.all()
    with open(file_key, 'w') as f:
        for user in users:
            json.dump(user.to_dict(), f)
            f.write('\n')

    s3.upload_file(file_key, bucket_name, file_key)
    return s3.generate_presigned_url('get_object', Params={'Bucket': bucket_name, 'Key': file_key}, ExpiresIn=3600)

def get_signed_download_url(bucket_name, file_key):
    s3 = boto3.client('s3')
    return s3.generate_presigned_url('get_object', Params={'Bucket': bucket_name, 'Key': file_key}, ExpiresIn=3600)

#### Pros

Allows the pipeline to process the data at its own pace.
Can be cached or pre-processed.

#### Cons

Involves additional storage and processing overhead.
May introduce delays if the file is large and needs to be uploaded.

#### Closing Summary

For an endpoint that handles millions of rows, streaming NDJSON offers the best balance between performance and complexity. It allows for efficient memory usage and real-time processing. However, pagination can be simpler but may require more memory. Generating a file plus a signed download URL provides flexibility but adds storage and upload overhead.

#### Common Mistakes/Follow-Ups
Misunderstanding Memory Usage: Pagination might lead to high memory consumption if not implemented carefully.
Improper Error Handling: Ensure robust error handling for streaming scenarios, especially in case of network issues or data corruption.
Security Concerns: When generating signed URLs, ensure proper access controls and secure key management.

**Comparison:**

```
Method	Pros	Cons
Pagination	Simple to implement	High memory usage, potential performance bottlenecks
Streaming NDJSON	Efficient memory usage, real-time processing	Requires downstream system support for streaming
File + Signed URL	Flexibility, caching options	Additional storage and upload overhead
```

**Code:**

```python
def get_users_page(page_size, page_number):
    start_index = (page_number - 1) * page_size
    end_index = start_index + page_size
    users = User.objects.all()[start_index:end_index]
    return [user.to_dict() for user in users]

def generate_users_stream():
    users = User.objects.all()
    for user in users:
        yield f"{json.dumps(user.to_dict())}\n"

import boto3

def create_user_file(bucket_name, file_key):
    s3 = boto3.client('s3')
    users = User.objects.all()
    with open(file_key, 'w') as f:
        for user in users:
            json.dump(user.to_dict(), f)
            f.write('\n')

    s3.upload_file(file_key, bucket_name, file_key)
    return s3.generate_presigned_url('get_object', Params={'Bucket': bucket_name, 'Key': file_key}, ExpiresIn=3600)

def get_signed_download_url(bucket_name, file_key):
    s3 = boto3.client('s3')
    return s3.generate_presigned_url('get_object', Params={'Bucket': bucket_name, 'Key': file_key}, ExpiresIn=3600)
```

**Example input / output:**

```text
# Example of generating a user stream
stream = generate_users_stream()
for line in stream:
    print(line)

# Example of creating and downloading a file
file_key = 'users.json'
bucket_name = 'my-bucket'
url = create_user_file(bucket_name, file_key)
print(url)
```

**Data:**

```json
{
  "users": [
    {"id": 1, "name": "Alice", "email": "alice@example.com"},
    {"id": 2, "name": "Bob", "email": "bob@example.com"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Endpoint Call
6. Pagination?
7. Return Paginated Data
8. Streaming NDJSON?
9. Stream Users
10. Generate File
11. Signed URL

